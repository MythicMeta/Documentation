# Sub-tasking / Task Callbacks

## What is sub-tasking?

Sub-tasking is the ability for a task to spin off sub-tasks and wait for them to finish before continuing execution of its own. Tasks will wait for all of their sub-tasks to complete before potentially entering a "submitted" state themselves for an agent to pick them up.

When a task has outstanding subtasks, its status will change to "delegating" while it waits for them all to finish.

<figure><img src="../../.gitbook/assets/Screenshot 2023-04-07 at 2.20.32 PM.png" alt=""><figcaption></figcaption></figure>

## What are task callbacks?

Task callbacks are functions that get executed when a task enters a "completed=True" state (i.e. when it completes successfully or encounters an error). These can be registered on a task itself

```python
async def create_go_tasking(self, taskData: MythicCommandBase.PTTaskMessageAllData) -> MythicCommandBase.PTTaskCreateTaskingMessageResponse:
    response = MythicCommandBase.PTTaskCreateTaskingMessageResponse(
        TaskID=taskData.Task.ID,
        CompletionFunctionName="formulate_output",
        Success=True,
    )
    return response
```

or on a subtask:

```python
await SendMythicRPCTaskCreateSubtask(
            MythicRPCTaskCreateSubtaskMessage(
                TaskID=taskData.Task.ID,
                CommandName="shell",
                Params="whoami",
                SubtaskCallbackFunction="formulate_output"
            )
        )
```

When Mythic calls these callbacks, it looks for the defined name in the command's `completed_functions` attribute like:

```
completion_functions = {"formulate_output": formulate_output}
```

Where the `key` is the same name of the function specified and the `value` is the actual reference to the function to call.

## Where are they?

Like everything else associated with a Command, all of this information is stored in your command's Python/GoLang file. Sub-tasks are created via RPC functions from within your command's `create_tasking` function (or any other function - i.e. you can issue more sub-tasks from within task callback functions). Let's look at what a callback function looks like:

```python
async def formulate_output( task: PTTaskCompletionFunctionMessage) -> PTTaskCompletionFunctionMessageResponse:
    # Check if the task is complete
    response = PTTaskCompletionFunctionMessageResponse(Success=True, TaskStatus="success")
    if task.TaskData.Task.Completed is True:
        # Check if the task was a success
        if not task.TaskData.Task.Status.includes("error"):
            # Get the interval and jitter from the task information
            interval = task.TaskData.args.get_arg("interval")
            jitter = task.TaskData.args.get_arg("interval")

            # Format the output message
            output = "Set sleep interval to {} seconds with a jitter of {}%.".format(
                interval / 1000, jitter
            )
        else:
            output = "Failed to execute sleep"

        # Send the output to Mythic
        resp = await SendMythicRPCResponseCreate(MythicRPCResponseCreateMessage(
                TaskID=taskData.Task.ID,
                Response=output.encode()
            ))

        if not resp.Success:
            raise Exception("Failed to execute MythicRPC function.")
    return response
```

## Task Callbacks

This is useful for when you want to do some post-task processing, actions, analysis, etc when a task completes or errors out. In the above example, the `formulate_output` function simply just displays a message to the user that the task is done. In more interesting examples though, you could use the `get_responses` RPC call like we saw above to get information about all of the output subtasks have sent to the user for follow-on processing.
