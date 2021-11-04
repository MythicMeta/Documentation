# Scripting Tasking

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.get_all_tasks()`:

```python
async def get_all_tasks(self) -> MythicResponse:
    """
    Get all of the tasks associated with the user's current operation
    :return: an array of Task objects
    """

async def get_all_tasks_for_callback(self, callback: Union[Callback, Dict]) -> MythicResponse:
    """
    Get the tasks (no responses) for a specific callback
    :param callback:
        if using the Callback class, the following must be set:
            id
    :return: an array of Task objects
    """

async def get_all_responses_for_task(self, task: Union[Task, Dict]) -> MythicResponse:
    """
    For the specified task, get all the responses
    :param task:
        if using the Task class, the following must be set:
            id
    :return: a Task object with Callback and an array of Responses filled in
    """

async def get_all_tasks_and_responses_grouped_by_callback(self) -> MythicResponse:
    """
    Get all tasks and responses for all callbacks in the current operation
    :return: an array of Callback objects with an array of Tasks and Responses each
    """
    
async def create_task(self, task: Task, return_on="preprocessing", timeout=None) -> MythicResponse:
    """
    Create a new task for a callback
    :param task:
        if using the Task class, the following must be set:
            callback: id
            command: cmd
            params
    :return: the Task in question
    """

```

This section also hooks into the following async callback functions:

```python
async def listen_for_all_tasks(self, callback_function=None, timeout=None):
    """
    Uses websockets to listen for all tasks within mythic for the current operation.
    :param callback_function: gets called on each notification
    :return:
    """

async def listen_for_new_tasks(self, callback_function=None, timeout=None):
    """
    Uses websockets to listen for all new tasks within mythic for the current operation.
    :param callback_function: gets called on each notification
    :return:
    """
```

The `listen_for_all_tasks` function gets `ALL` tasks in the operation and continue to listen for new ones. This is slightly different than the `listen_for_new_tasks` function which simply starts listening for new tasking and doesn't give historic data.

## Creating Tasks

Most functions in this section are pretty straight forward, but creating new Tasks can be complicated. To help with this, let's look at some examples:

```python
task = mythic_rest.Task(callback=callback['id'], command=mythic_rest.Command(cmd="shell"), params="whoami")
submit = await mythic.create_task(task, return_on="submitted")
await mythic_rest.json_print(submit)
print("task is submitted, now to wait for responses to process")
results = await mythic.gather_task_responses(submit.response.id, timeout=20)
print("got array of results of length: " + str(len(results)))
```

the `create_task` function takes in the Task you want to create. At a simple example, let's look at issuing the `shell whoami` command that you'd typically type out on the command line. We need to specify a few things:

```python
mythic_rest.Task(callback=callback['id'], command=mythic_rest.Command(cmd="shell"), params="whoami")
```

We have to specify the callback (this can be either with a `Callback` object or just the `id` associated with the callback). This tells Mythic which callback we're wanting to interact with. Then we need to specify the command we want to issue. This can be either a `Command` object, or simply the string of the name of the command we want to execute, `shell` in this instance. The last piece here is specifying the parameters we need to send down. `params` will either be a string or you can specify the JSON associated with the command as well. In this case, the parameters for our `shell` command is simply the string `whoami`. At this point, we've described the Task we want to create. Now we can issue it with `mythic.create_task`.&#x20;

`create_task` takes an interesting parameter though - `return_on`. This specifies when you want to return from this function call. If you've used the interface before, you've noticed that as you go through the tasking life cycle, the status changes between a few status - `preprocessing`, `submitted`, `processing`, `processed`, `completed`, `error`, and sometimes `building...`. The `return_on` function allows you to specify when you're ready to return. For example, if you don't specify this, the function will return as soon as Mythic gets your RESTful request (i.e. your tasking is in the `preprocessing` status). The different status types mean:

* `preprocessing` - Mythic got the Task request and set it off to the payload type's Docker container
* `submitted` - Everything went well with creating Tasking and it's ready for an agent to pick up
* `processing` - An agent picked up the tasking, but hasn't returned anything
* `processed` - An agent sent back _at least one_ response, but hasn't indicated that the tasking is done
* `completed` - An agent indicated that the tasking is completed
* `error` - An agent indicated that something went wrong after picking up the tasking.

Regardless of the status you say to `return_on`, if the task switches to `completed` or `error`, then it will return. You can also specify a `timeout` in seconds of how long to wait for your status to match. This is helpful for scripting so that you don't wait indefinitely for a `completed` status if your agent is dead for example.

In the above example, we simply return on `submitted`, so there are no responses yet. In this case, we only want to continue on with our function when we have _all_ of the tasking responses, not when we're only partially done. To facilitate this, we have an additional helper function:&#x20;

```python
await mythic.gather_task_responses(submit.id, timeout=20)
```

This function takes a task id and how long we're willing to wait. This returns an array of the responses (so if there's just one response, it'll be an array of one).

### Creating tasks with files

All of the above is fine for most tasking, but what about the case where you want to upload a file with your tasking, such as with the `upload` command. Let's take that as an example. The `apfell` upload command takes two parameters:

* `remote_path` - a parameter of type `String` that indicates the remote path of where the file will be uploaded to
* `file` - a parameter of type `File` that indicates the actual file we upload. If you do this in the UI, then you'll se a popup modal with a button for you to select a file from disk. Obviously, this isn't available for scripting, so we need to do something else.&#x20;

```python
task = mythic_rest.Task(callback=6, 
    command=mythic_rest.Command(cmd="upload"), 
    params={"remote_path":"test.txt"}, 
    files=[
        mythic_rest.TaskFile(content=b'TEST CONTENT', 
                 filename="originalfile.yo", 
                 param_name="file")
          ]
      )
submit = await mythic.create_task(task)
```

The above example is mostly the same as the `issue_shell_whoami` example above, except for one addition - the `files` parameter. This is an array of `TaskFile` objects. Each `TaskFile` object takes in a few parameters:

* `content` - this is the binary data that we're trying to upload
* `filename` - this is the filename we want associated with this binary content in the UI
* `param_name` - this is the name of the parameter we're referencing with the file content. In our `upload` example, this parameter was simply called `file`, so that's what you see here. In other commands, that parameter could be called anything though.

Since you can potentially upload multiple files for a given command, this `files` parameter is an array of entries. Otherwise, you call the `mythic.create_task` just like any other tasking function.
