# OPSEC Checking

## What is OPSEC Checking

It's often useful to perform some operational security checks before issuing a task based on everything you know so far, or after you've generated new artifacts for a task but before an agent picks it up. This allows us to be more granular and context aware instead of the blanket command blocking that's available from the Operation Management page in Mythic.

## Where is OPSEC Checking?

OPSEC checks and information for a command is located in the same file where everything else for the command is located. We'll be tracking this data with a new class called `CommandOPSEC` . Let's take an example all the way through:

```python
class ShellOPSEC(CommandOPSEC):
    injection_method = ""
    process_creation = "/bin/bash -c"
    authentication = ""

    async def opsec_pre(self, task: MythicTask):

        processes = await MythicResponseRPC(task).search_database(
            table="process",
            search={"host": task.callback.host}
        )
        if len(processes.response) == 0:
            task.opsec_pre_blocked = True
            task.opsec_pre_message = f"This spawns {self.process_creation} and there is no process data on the host yet."
            task.opsec_pre_message += "\nRun \"list_apps\" first to check for dangerous processes"
            task.opsec_pre_bypass_role = "operator"
            return
        else:
            processes = await MythicResponseRPC(task).search_database(
                table="process",
                search={"name": "Microsoft Defender", "host": task.callback.host}
            )
            if len(processes.response) > 0:
                task.opsec_pre_blocked = True
                task.opsec_pre_message = f"Microsoft Defender spotted on the host in running processes. Don't spawn commands this way"

    async def opsec_post(self, task: MythicTask):
        processes = await MythicResponseRPC(task).search_database(
            table="process",
            search={"name": "Microsoft Defender", "host": task.callback.host}
        )
        if len(processes.response) > 0:
            task.opsec_post_blocked = True
            task.opsec_post_message = f"Microsoft Defender spotted on the host in running processes. Really, don't do this"
            
class ShellCommand(CommandBase):
    cmd = "shell"
    needs_admin = False
    help_cmd = "shell {command}"
    description = """This runs {command} in a terminal by leveraging JXA's Application.doShellScript({command}).
WARNING! THIS IS SINGLE THREADED, IF YOUR COMMAND HANGS, THE AGENT HANGS!"""
    version = 1
    author = "@its_a_feature_"
    attackmapping = ["T1059"]
    argument_class = ShellArguments
    opsec_class = ShellOPSEC
```

{% hint style="info" %}
Notice that we hook this OPSEC class instance into our `Shell` command by adding a new parameter called `opsec_class` and pointing it at our new OPSEC class
{% endhint %}

### opsec\_pre / opsec\_post

In the case of doing operational checks before a task's `create_tasking` is called, we have the `opsec_pre` function. Similarly, the `opsec_post` function happens after your `create_tasking`, but before your task is finally ready for an agent to pick it up.

These functions don't return anything - instead, they modifies properties on the task itself. Specifically, the function can set:

* `opsec_pre/post_blocked` - this indicates True/False for if the function decides the task should be blocked or not
* `opsec_pre/post_message` - this is the message to the operator about the result of doing this OPSEC check
* `opsec_pre/post_bypass_role` - this determines who should be able to bypass this check. The default is `lead` to indicate that only the lead of the operation should be able to bypass it, but you can set it to `operator` to allow any operators to bypass the check. This is helpful in cases where it's not necessarily a "block", but something you want to make sure operators acknowledge as a potential security risk

This function takes one argument, the task itself. Just like with the `create_tasking` function, you get a LOT of context with the `task` variable ([Create\_Tasking](create\_tasking.md#rpc-functionality)).

As the name of the functions imply, the `opsec_pre` check happens before `create_tasking` function runs and the `opsec_post` check happens after the `create_tasking` function runs. If you set `opsec_pre_blocked` to True, then the `create_tasking` function isn't executed until an approved operator bypasses the check. Then, execution goes back to `create_tasking` and the `opsec_post`. If that one also sets blocked to True, then it's again blocked at the user to bypass it. At this point, if it's bypassed, the task status simply switched to `Submitted` so that an agent can pick up the task on next checkin.

## OPSEC Information Tracking

As part of the `CommandOPSEC` class, there are a few parameters in addition to the `opsec_pre` and `opsec_post` functions. These pieces of information allow us to start tracking what exactly the command is going to do from an OPSEC perspective. Does it inject into another process? Does it spawn processes? Is it going to do any specific kind of authentication (network, new credential, making a token, etc)?

By tracking this information as part of this OPSEC class, we can start making broader checks and more generalized functions to call. Maybe for every command that does process injection, we want to first (as part of an `opsec_pre` check) check for certain defensive products on the box that might pick up on that type of activity. There's no reason for us to re-write that every time in each function. That leads to fragmented and repeated code. We might also want to check to see if there are any currently running tasks for a specific host that are doing certain kinds of authentication before we allow the current task to inject a kerberos ticket or steal a token.

## OPSEC Scripting

From the `opsec_pre` and `opsec_post` functions, you have access to the entire task/callback information like you do in [Create\_Tasking](create\_tasking.md#available-context). Additionally, you have access to the entire RPC suite just like in [Create\_Tasking](create\_tasking.md#rpc-functionality).
