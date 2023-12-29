---
description: Manipulate tasking before it's sent to the agent
---

# 4. Create Tasking

## Create Tasking

All commands must have a create\_go\_tasking function with a base case like:



{% tabs %}
{% tab title="Python" %}
```python
async def create_go_tasking(self, taskData: MythicCommandBase.PTTaskMessageAllData) -> MythicCommandBase.PTTaskCreateTaskingMessageResponse:
    response = MythicCommandBase.PTTaskCreateTaskingMessageResponse(
        TaskID=taskData.Task.ID,
        Success=True,
    )
    return response
```

{% hint style="info" %}
`create_go_tasking` is new in Mythic v3.0.0. Prior to this, there was the `create_tasking` function. The new change supports backwards compatibility, but the new function provides a lot more information and structured context that's not available in the `create_tasking` function. The `create_go_tasking` function also mirrors the GoLang's `create_tasking` function.
{% endhint %}
{% endtab %}

{% tab title="Golang" %}
```go
TaskFunctionCreateTasking: func(taskData *agentstructs.PTTaskMessageAllData) agentstructs.PTTaskCreateTaskingMessageResponse {
	response := agentstructs.PTTaskCreateTaskingMessageResponse{
		Success: true,
		TaskID:  taskData.Task.ID,
	}
	return response
},
```
{% endtab %}
{% endtabs %}

When an operator types a command in the UI, whatever the operator types (or whatever is populated based on the popup modal) gets sent to this function after the input is parsed and validated by the TaskArguments and CommandParameters functions mentioned in [Commands](../adding-commands/commands.md).

It's here that the operator has full control of the task before it gets sent down to an agent. The task is currently in the "preprocessing" stage when this function is executed and allows you to do many things via Remote Procedure Calls (RPC) back to the Mythic server.

{% hint style="info" %}
A graphical flow of what goes on is here: [tasking flow](../../../message-flow/operator-submits-tasking.md).
{% endhint %}

### Available Context

So, from this create tasking function, what information do you immediately have available? [https://github.com/MythicMeta/MythicContainerPyPi/blob/main/mythic\_container/MythicCommandBase.py#L1071-L1088](https://github.com/MythicMeta/MythicContainerPyPi/blob/main/mythic\_container/MythicCommandBase.py#L1071-L1088) <-- this class definition provides the basis for what's available.&#x20;

* `taskData.Task` - Information about the Task that's issued
* `taskData.Callback` - Information about the Callback for this task
* `taskData.Payload` - Information about the packing payload for this callback
* `taskData.Commands` - A list of the commands currently loaded into this callback
* `taskData.PayloadType` - The name of this payload type
* `taskData.BuildParameters` - The build parameters and their values used when building the payload for this callback
* `taskData.C2Profiles` - Information about the C2 Profiles included inside of this callback.
* `taskData.args` - access to the associated arguments class for this command that already has all of the values populated and validated. Let's say you have an argument called "remote\_path", you can access it via `taskData.args.get_arg("remote_path")` .
  * Want to change the value of that to something else? `taskData.args.add_arg("remote_path", "new value")`.
  * Want to change the value of that to a different type as well? `taskData.args.add_arg("remote_path", 5, ParameterType.Number)`
  * Want to add a new argument entirely for this specific instance as part of the JSON response? `taskData.args.add_arg("new key", "new value")`. The `add_arg` functionality will overwrite the value if the key exists, otherwise it'll add a new key with that value. The default ParameterType for args is `ParameterType.String`, so if you're adding something else, be sure to change the type. **Note**: If you have multiple parameter groups as part of your tasking, make sure you specify _which_ parameter group your new argument belongs to. By default, the argument gets added to the `Default` parameter group. This could result in some confusion where you add an argument, but it doesn't get picked up and sent down to the agent.
  * You can also remove args `taskData.args.remove_arg("key")`, rename args `taskData.args.rename_arg("old key", "new key")`
  * You can also get access to the user's commandline as well via `taskData.args.commandline`
  * Want to know if an arg is in your args? `taskData.args.has_arg("key")`
* `taskData.Task.TokenID` - information about the token that was used with the task. This requires that the callback has at some point returned tokens for Mythic to track, otherwise this will be 0.

In the `PTTaskCreateTaskingMessageResponse`, you can set a variety of attributes to reflect changes back to Mythic as a result of your processing: [https://github.com/MythicMeta/MythicContainerPyPi/blob/main/mythic\_container/MythicCommandBase.py#L820](https://github.com/MythicMeta/MythicContainerPyPi/blob/main/mythic\_container/MythicCommandBase.py#L820)

* `Success` - did your processing succeed or not? If not, set `Error` to a string value representing the error you encountered.
* `CommandName` - If you want the agent to see the command name for this task as something _other_ than what the actual command's name is, reflect that change here. This can be useful if you are creating an alias for a command. So, your agent has the command `ls`, but you create a script\_only command `dir`. During the processing of `dir` you set the `CommandName` to `ls` so that the agent sees `ls` and processes it as normal.
* `TaskStatus` - If something went wrong and you want to reflect a specific status to the user, you can set that value here. Status that start with `error:` will appear `red` in the UI.&#x20;
* `Stdout` and `Stderr` - set these if you want to provide some additional stdout/stderr for the task but don't necessarily want it to clutter the user's interface. This is helpful if you're doing additional compliations as part of your tasking and want to store debug or error information for later.
* `Completed` - If this is set to `True` then Mythic will mark the task as done and won't allow an agent to pick it up.
*   `CompletionFunctionName` - if you want to have a specific local function called when the task completes (such as to do follow-on tasking or more RPC calls), then specify that function name here. This requires a matching entry in the command's `completion_functions` like follows:&#x20;

    ```
    completion_functions = {"formulate_output": formulate_output}
    ```
* `ParameterGroupName` - if you want to explicitly set the parameter group name instead of letting Mythic figure it out based on which parameters have values, you can specify that here.&#x20;
* `DisplayParams` - you can set this value to a string that you'd want the user to see instead of the `taskData.Task.OriginalParams`. This allows you to leverage the JSON structure of the popup modals for processing, but return a more human-friendly version of the parameters for operators to view. There's a new menu-item in the UI when viewing a task that you can select to view all the parameters, so on a case-by-case basis an operator can view the original JSON parameters that were sent down, but this provides a nice way to prevent large JSON blobs that are hard to read for operators while still preserving the nice JSON scripting features on the back-end.

### RPC Functionality

This additional functionality is broken out into a series of files ([https://github.com/MythicMeta/MythicContainerPyPi/tree/main/mythic\_container/MythicGoRPC](https://github.com/MythicMeta/MythicContainerPyPi/tree/main/mythic\_container/MythicGoRPC)) file that you can import at the top of your Python command file.&#x20;

They all follow the same format:&#x20;

```python
async def SendMythicRPC*(MythicRPC*Message) -> MythicRPC*MessageResponse
```
