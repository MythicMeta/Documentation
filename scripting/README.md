---
description: How to use the Scripting API
---

# Scripting

## What is Mythic scripting?

The main Mythic server Docker container is based around WebSockets and RESTful API endpoints. We can hit the same RESTful endpoints and listen to the same WebSocket endpoints that the main browser user interface uses as part of scripting, which means scripting can technically be done in any language.&#x20;

## Where is it?

Install the PyPi package via pip `pip3 install mythic` . The current mythic package is version `0.0.23` and reports to Mythic as scripting version "3". The code for it is public - [https://github.com/MythicMeta/Mythic\_Scripting](https://github.com/MythicMeta/Mythic\_Scripting)

The RESTful interfaces that mythic currently uses are located under `mythic_rest`, so do the following:

```python
from mythic import mythic_rest
mythic = mythic_rest.Mythic(username="mythic_admin", 
                    password="mythic_password",
                    server_ip="192.168.205.151", 
                    server_port="7443", 
                    ssl=True, 
                    global_timeout=-1)
```

The PyPI package specifically refers to `mythic_rest` because there are likely to be other interfaces in the future (such as graphql).

### Scripting base

The scripting base is pretty simple:

```python
from mythic import mythic_rest
from sys import exit


async def scripting():
    # sample login
    mythic = mythic_rest.Mythic(username="mythic_admin", 
                    password="mythic_password",
                    server_ip="192.168.205.151", 
                    server_port="7443", 
                    ssl=True, 
                    global_timeout=-1)
    await mythic.login()
    # either set an api token to use or create a new one to start using
    resp = await mythic.set_or_create_apitoken()
    await mythic_rest.json_print(resp)
    
async def main():
    await scripting()
    try:
        while True:
            pending = asyncio.Task.all_tasks()
            if len(pending) == 0:
                exit(0)
            else:
                await asyncio.gather(*pending)
    except KeyboardInterrupt:
        pending = asyncio.Task.all_tasks()
        for t in pending:
            t.cancel()
loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

Line 1 imports all of the functions from the mythic api into the script. From here, there's two functions we write:

1. `main()` - this function calls the scripting function asynchronously ( `await scripting()` ), then loops through all functions sitting on the current event loop until everything is done. This is what allows you to eventually hook into the WebSocket eventing and wait for notifications.
2. `scripting()` - this is the initial function where you write your code.&#x20;

In the scripting function, we do a few things:

1. The first thing is to create an instance of the `Mythic` server. This includes the credentials we use to log in and information about the server itself. The `global_timeout` is an optional parameter to globally provide timeouts if no others are provided. If the `global_timeout` is set to anything less than 0, then the program will wait indefinitely.
2. Once this object is created, the next thing is to log into mythic with `await mythic.login()`. This sends the credentials over the connection and gets back the standard JWT access\_token and refresh\_tokens.&#x20;
3. The next standard thing to do is to do `await mythic.set_or_create_apitoken()`. Because dealing with JWT access tokens is annoying (they have short timestamps) and trying to make sure you properly deal with refresh tokens in scripting is error prone, there's a helper function to create a user level API token. These API tokens show up in your `settings` page and can be deactivated or deleted. The advantage of these tokens is that while they're active, they don't expire. This is very useful for long running tasks. Doing `set_or_create_apitoken()` is a helper function to get and potentially create a user-level API token and set it on your current `mythic` instance.&#x20;

If you've already created an API token that you'd like to use instead of supplying your username and password, you can create the `mythic` object with the following:

```python
mythic = mythic_rest.Mythic(server_ip="192.168.205.151", server_port="7443", ssl=True,
                apitoken="ej..api token here")
```

Because you already have the apitoken set, there's no need for additional calls to `login` or `set_or_create_apitoken`.&#x20;

### Scripting Principals

The Mythic scripting file tries to make everything into classes/objects so that it's easier to use with an IDE than just python dictionaries everywhere. This makes it closely resemble the Mythic database (although there's a lot more to the database than what's captured in the scripting so far).

Everything call to mythic that doesn't start with `listen` will return a `MythicResponse` class object. This allows you to properly inspect the `.status` of the query ( `success` or `error`) as well as the `.response_code` (the web response code from the query). The actual response object(s) is in the `.response` component. For example:

```python
resp = await mythic.get_all_tasks() # this returns a MythicResponse object
# resp.response is an array of Task objects
await mythic_rest.json_print(resp) # print out the MythicResponse object as JSON
for t in resp.response:
    await mythic_rest.json_print(x) # print the Task object as JSON
```

the `json_print` function allows you to easily print Mythic class objects as JSON. If you just want to access it as JSON data (i.e. not printing it), then each object has a `.to_json()` function.&#x20;

### Listening for Notifications

So far, the components have been doing RESTful API calls directly. But one of the more interesting aspects of scripting is to listen for specific notification events and reacting to them. In Mythic, these are all indicated with function names starting with `listen_for`. These functions all have the same general format:

```python
async def listen_for_new_callbacks(self, callback_function=None, timeout=None):
        """
        Uses websockets to listen for all notifications related new callbacks.
        To stop listening, call cancel() on the result from this function call
        :param callback_function: gets called on each notification
        :return:
        """
        url = "{}{}:{}/ws/new_callbacks/current_operation".format(self._ws, self._server_ip, self._server_port)
        if callback_function:
            task = await self.stream_output(url, callback_function, timeout)
        else:
            task = await self.stream_output(url, self.print_websocket_output, timeout)
        return task
```

If a `callback_function` isn't supplied, then the resulting data is simply printed to the screen. The timeout can be used for indicating how long you want to listen.  Let's see how this can be used:

```python
from mythic import mythic_rest
from sys import exit
import asyncio


async def scripting():
    # sample login
    mythic = mythic_rest.Mythic(username="mythic_admin", password="mythic_password",
                    server_ip="192.168.205.151", server_port="7443", ssl=True, global_timeout=-1)
    await mythic.login()
    # either set an api token to use or create a new one to start using
    resp = await mythic.set_or_create_apitoken()
    await mythic.listen_for_new_callbacks(issue_shell_whoami)

async def issue_shell_whoami(mythic, callback):
    try:
        print("in issue_shell_whoami, about to create a task")
        task = mythic_rest.Task(callback=callback, command=mythic_rest.Command(cmd="shell"), params="whoami")
        submit = await mythic.create_task(task, return_on="submitted")
        await mythic_rest.json_print(submit)
        print("task is submitted, now to wait for responses to process")
        results = await mythic.gather_task_responses(submit.id, timeout=20)
        print("got array of results of length: " + str(len(results)))
    except Exception as e:
        print(str(e))

async def main():
    await scripting()
    try:
        while True:
            pending = asyncio.Task.all_tasks()
            if len(pending) == 0:
                exit(0)
            else:
                await asyncio.gather(*pending)
    except KeyboardInterrupt:
        pending = asyncio.Task.all_tasks()
        for t in pending:
            t.cancel()
loop = asyncio.get_event_loop()
loop.run_until_complete(main())

```

This example waits for new callbacks, and when a new one is recognized by Mythic, the `issue_shell_whoami` function defined on lines 14-25 is executed. These callback functions always have 2 parameters - the mythic instance associated with the data and a `STRING` representation of the data. To get back to a python dictionary, run `json.loads(data)`. At this point, you can either deal with data as a dictionary directly (as shown in the above code), or you can cast it back to the right object. In the above example, we could do `callback = Callback(**json.loads(data))` which would result in `callback` being a `Callback` object instead of a python dictionary.&#x20;

## Troubleshooting

When scripting, you might run into issues, so it's important to know how to troubleshoot what's going on. Every function returns a MythicResponse object. All Mythic specific objects (Responses, Tasks, Operators, etc) can be printed with `await mythic_rest.json_print(object here)`. This provides a clean way to see the output that you actually got back. The reason everything returns a `MythicResponse` instead of just the object is so that we can capture HTTP response codes, raw output, _and_ the Object casted types. This allows you to more easily manipulate and see data you get back via scripting.

