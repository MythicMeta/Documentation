# OPSEC Checking

## What is OPSEC Checking

It's often useful to perform some operational security checks before issuing a task based on everything you know so far, or after you've generated new artifacts for a task but before an agent picks it up. This allows us to be more granular and context aware instead of the blanket command blocking that's available from the Operation Management page in Mythic.

## Where is OPSEC Checking?

OPSEC checks and information for a command is located in the same file where everything else for the command is located. Let's take an example all the way through:

```python
class ShellCommand(CommandBase):
    cmd = "shell"
    needs_admin = False
    help_cmd = "shell {command}"
    description = """This runs {command} in a terminal by leveraging JXA's Application.doShellScript({command}).
WARNING! THIS IS SINGLE THREADED, IF YOUR COMMAND HANGS, THE AGENT HANGS!"""
    version = 1
    author = "@its_a_feature_"
    attackmapping = ["T1059", "T1059.004"]
    argument_class = ShellArguments
    attributes = CommandAttributes(
        suggested_command=True
    )

    async def opsec_pre(self, taskData: PTTaskMessageAllData) -> PTTTaskOPSECPreTaskMessageResponse:
        response = PTTTaskOPSECPreTaskMessageResponse(
            TaskID=taskData.Task.ID, Success=True, OpsecPreBlocked=True,
            OpsecPreBypassRole="other_operator",
            OpsecPreMessage="Implemented, but not blocking, you're welcome!",
        )
        return response

    async def opsec_post(self, taskData: PTTaskMessageAllData) -> PTTTaskOPSECPostTaskMessageResponse:
        response = PTTTaskOPSECPostTaskMessageResponse(
            TaskID=taskData.Task.ID, Success=True, OpsecPostBlocked=True,
            OpsecPostBypassRole="other_operator",
            OpsecPostMessage="Implemented, but not blocking, you're welcome! Part 2",
        )
        return response

    async def create_go_tasking(self, taskData: MythicCommandBase.PTTaskMessageAllData) -> MythicCommandBase.PTTaskCreateTaskingMessageResponse:
        response = MythicCommandBase.PTTaskCreateTaskingMessageResponse(
            TaskID=taskData.Task.ID,
            Success=True,
        )
        await SendMythicRPCArtifactCreate(MythicRPCArtifactCreateMessage(
            TaskID=taskData.Task.ID, ArtifactMessage="{}".format(taskData.args.get_arg("command")),
            BaseArtifactType="Process Create"
        ))

        response.DisplayParams = taskData.args.get_arg("command")
        return response
```

### opsec\_pre / opsec\_post

In the case of doing operational checks before a task's `create_tasking` is called, we have the `opsec_pre` function. Similarly, the `opsec_post` function happens after your `create_tasking`, but before your task is finally ready for an agent to pick it up.

* `opsec_pre/post_blocked` - this indicates True/False for if the function decides the task should be blocked or not
* `opsec_pre/post_message` - this is the message to the operator about the result of doing this OPSEC check
* `opsec_pre/post_bypass_role` - this determines who should be able to bypass this check. The default is `lead` to indicate that only the lead of the operation should be able to bypass it, but you can set it to `operator` to allow any operators to bypass the check. You can also set this to `other_operator` to indicate that somebody _other_ than the operator that issued the task must approve it. This is helpful in cases where it's not necessarily a "block", but something you want to make sure operators acknowledge as a potential security risk

As the name of the functions imply, the `opsec_pre` check happens before `create_tasking` function runs and the `opsec_post` check happens after the `create_tasking` function runs. If you set `opsec_pre_blocked` to True, then the `create_tasking` function isn't executed until an approved operator bypasses the check. Then, execution goes back to `create_tasking` and the `opsec_post`. If that one also sets blocked to True, then it's again blocked at the user to bypass it. At this point, if it's bypassed, the task status simply switched to `Submitted` so that an agent can pick up the task on next checkin.

## OPSEC Scripting

From the `opsec_pre` and `opsec_post` functions, you have access to the entire task/callback information like you do in [Create\_Tasking](create\_tasking.md#available-context). Additionally, you have access to the entire RPC suite just like in [Create\_Tasking](create\_tasking.md#rpc-functionality).
