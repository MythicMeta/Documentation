---
description: Manipulate tasking before it's sent to the agent
---

# Create\_Tasking

## create\_tasking

All commands must have a create\_tasking function with a base case like:

```python
async def create_tasking(self, task: MythicTask) -> MythicTask:
    return task
```

When an operator types a command in the UI, whatever the operator types (or whatever is populated based on the popup modal) gets sent to this function after the input is parsed and validated by the TaskArguments and CommandParameters functions mentioned in [Commands](commands.md).&#x20;

It's here that the operator has full control of the task before it gets sent down to an agent. The task is currently in the "preprocessing" stage when this function is executed and allows you to do many things via Remote Procedure Calls (RPC) back to the Mythic server.&#x20;

### Available Context

So, from this create\_tasking function, what information do you immediately have available?

* `task.task_id` - the task number associated with this task (not the UUID the agent sees, the integer value that operators see)
* `task.agent_task_id` - the UUID associated with this task that an agent sees
* `task.original_params` - the string of original parameters that was passed to this function before anything was swapped out for default values or edited.
* `task.completed` - boolean indicating if the task is marked as completed or not
* `task.operator` - the name of the operator that issued the task
* `task.status` - the current status of the task (defaults to MythicStatus.Success).&#x20;
  * Options are `MythicStatus.Error, MythicStatus.Completed`, `MythicStatus.Processed`, `MythicStatus.Processing`
  * If you set the status to `Success`, then Mythic will set the task to `Submitted` so that it's ready to be picked up by the agent. If you set it to error, completed, processed, or processing, then the agent _won't_ pick it up.
* `task.callback` - information about the task's associated callback. This has a LOT of information, so let's break it down more:
  * `task.callback.build_parameters` - this is a dictionary of all the build parameters used to generate the payload that the callback is based on. This is the same sort of information from [Payload Type Info](payload-type-info.md#build-parameters). So, if you had a build parameter called "version", you could see that associated value with `task.callback.build_parameters.['version']`
  * `task.callback.c2info` - this is an array of dictionaries of the different c2 profiles built into the callback. Just like [Payload Type Info](payload-type-info.md#building) where you iterate over them, you can do the same here. For example, to see a parameter called "callback\_host" in our first c2 profile is `task.callback.c2info[0]['callback_host']`.&#x20;
  * `task.callback.user` is the user for the callback
  * `task.callback.host` is the hostname of the callback
  * ... These match up with all the fields used for doing the initial callback - [Action: Checkin](../c2-related-development/c2-profile-code/agent-side-coding/initial-checkin.md#plaintext-checkin)
    * Something to note about `integrity_level` values - these are based on Windows standard integrity levels where 2 == normal account, 3 == high integrity (or `root`), and 4 == SYSTEM (but in most cases for operators, 3 and 4 are the same).
* `task.args` - access to the associated arguments class for this command that already has all of the values populated and validated. Let's say you have an argument called "remote\_path", you can access it via `task.args.get_arg("remote_path")` .&#x20;
  * Want to change the value of that to something else? `task.args.add_arg("remote_path", "new value")`.&#x20;
  * Want to change the value of that to a different type as well? `task.args.add_arg("remote_path", 5, ParameterType.Number)`
  * Want to add a new argument entirely for this specific instance as part of the JSON response? `task.args.add_arg("new key", "new value")`. The `add_arg` functionality will overwrite the value if the key exists, otherwise it'll add a new key with that value. The default ParameterType for args is `ParameterType.String`, so if you're adding something else, be sure to change the type.
  * You can also remove args `task.args.remove_arg("key")`, rename args `task.args.rename_arg("old key", "new key")`
  * You can also get access to the user's commandline as well via `task.args.commandline`
  * Want to know if an arg is in your args? `task.args.has_arg("key")`
* `task.token` - information about the token that was used with the task. This requires that the callback has at some point returned tokens for Mythic to track, otherwise this will be NULL.
* `task.opsec_*` - these opsec\* components come into play before and after the `create_tasking` function is actually called. These are broken out into two main pieces - `opsec_pre_*` and `opsec_post_*`. Each one tracks:
  * `_blocked` - indicates if the corresponding opsec checking function blocked the operator
  * `_message` - the associated message after running the corresponding opsec checking function
  * `_bypassed` - if a user bypassed the check anyway
  * `_bypass_role` - who is allowed to bypass the check, any operator or just the lead
  * `_bypass_user` - the user who ended up bypassing the check
* `task.set_stdout(string)` - Allows you to provide some task-specific standard output that you can then query and view at a later time through the Mythic UI. This helps if you need to go back and debug a task.
* `task.set_stderr(string)` - Allows you to provide some task-specific standard error output that you can then query and view at a later time through the Mythic UI. This helps if you need to go back and debug a task.
* `task.display_params` - you can set this value to a string that you'd want the user to see instead of the `task.original_params`. This allows you to leverage the JSON structure of the popup modals for processing, but return a more human-friendly version of the parameters for operators to view. There's a new menu-item in the UI when viewing a task that you can select to view all the parameters, so on a case-by-case basis an operator can view the original JSON parameters that were sent down, but this provides a nice way to prevent large JSON blobs that are hard to read for operators while still preserving the nice JSON scripting features on the back-end.

### RPC Functionality

This additional functionality is broken out into a MythicRPC file that you can import at the top of your Python command file. You can pull the current set of available functions, their prototypes, their args, and their docstrings by executing the `get_functions` method:

```python
from mythic_payloadtype_container.MythicRPC import *
import sys
resp = await MythicRPC().get_functions()
print(resp.response)
sys.stdout.flush()
```

Once you have that set of functions, they are all callable via the same specific format:

```python
from mythic_payloadtype_container.MythicRPC import *
response = await MythicRPC().execute("function name", argument1=val, argument2=val, ...etc)
```

And they all return a ReponseRPC object with a few attributes:

```python
from mythic_payloadtype_container.MythicRPC import *
response = await MythicRPC().execute("function name", argument1=val, argument2=val, ...etc)
response.status == MythicStatus.Success
response.response # (this is the JSON response you get back from Mythic)
response._raw_response # this is the raw response you get back that's then parsed into the status and response fields
```

#### All RPC Functions

All RPC functions are available in the [MythicRPC](../../updating/mythic-2.1-greater-than-2.2-updates/agents-2.1.-greater-than-2.2.2/mythicrpc.md) page, but some of them are called out below from a higher level standpoint on how you might use them:

* `create_file` - Register a file in the Mythic database and get back a file UUID. This UUID allows an agent to chunk files as it sends them from Mythic -> agent and allows the file to be tracked by Mythic.
* `get_file` - Search the current operation for a file that hasn't been deleted with the current name, if it exists, return the contents of the file and all metadata about it. This allows you to provide short-hand names on the command-line for an operator, but still translate that into a file UUID that the agent/Mythic can track.
* `delete_file_browser` - Mark the files in the `files` parameter as "removed" from the file browser. The `files` parameter takes in the same format as in [File Browser](../hooking-features/file-browser.md#agent-file-removal-responses)
* `create_file_browser` - add files to the file browser. This takes in the same format of information as would be returned from [File Browser](../hooking-features/file-browser.md#agent-file-browsing-responses)
* `get_payload` - Search the current operator for a payload with a specific UUID, if it exists, return it and the contents of the associated file. If there's a command parameter type of "Payload" and the user selects something you don't want or the Payload Type is right, but there's a parameter used that's incompatible with the tasking, this is where you're able to find that information.
* `create_payload_on_host` - Register a payload's UUID with a specific host. This informs Mythic that somehow, a specific payload is on a specific host. This allows Mythic to do some back-end auto tabulation and make more complete dialogs when operators then try to link to specific payloads.
* `create_payload_from_uuid` - Given a payload's UUID, ask Mythic to kick off the creation of a new payload based on the parameters and information associated with the given UUID. Since payload creation is an asynchronous task with potentially other Docker containers, there's a bit of a catch here. When doing this, you will get back the UUID of the new payload. You have to then loop within your tasking function to repeatedly call the `get_payload` function to see when the creation process is done (successfully or if it errored). When creating the payload, if you specify the additional parameter of `destination_host`, then Mythic will track that the newly created payload exists on that host. This will allow you to automatically populate payloads for doing P2P connections.
*   `create_payload_from_parameters` - Task Mythic to make a new payload from explicit parameters. When creating the payload, if you specify the additional parameter of `destination_host`, then Mythic will track that the newly created payload exists on that host. This will allow you to automatically populate payloads for doing P2P connections.

    This includes the payload type, list of commands, build parameters, etc.&#x20;

    * The C2 profile list takes a specific format:
      * \[{ "c2\_profile": "HTTP", "c2\_profile\_parameters": { "param\_name": value, "param": value} }]
    * The build parameter list also takes a specific format:
      * \[ {"name": "param name", "value": "param value"} ]
*   `build_payload_from_MythicPayloadRPCResponse` - If you got information about a payload via the `get_payload_by_uuid` and wanted to make a new payload from that, but with a few adjustments, then this function allows you to do just that. The following is an example of getting a payload based on a template, modifying a c2 parameter value, and tasking a build. When creating the payload, if you specify the additional parameter of `destination_host`, then Mythic will track that the newly created payload exists on that host. This will allow you to automatically populate payloads for doing P2P connections.

    There are two helper functions on `MythicPayloadRPCResponses` for adjusting a few key parameters:

    * `set_profile_parameter_value` - give the name of a c2 profile, parameter name, and a new value - this function makes that replacement or addition.
    * `set_build_parameter_value` - give the name of a build parameter name and value - this function makes the replacement or addition.

```python
async def create_tasking(self, task: MythicTask) -> MythicTask:
    orig = await MythicPayloadRPC(task).get_payload_by_uuid(task.args.get_arg("template"))
    orig.set_profile_parameter_value("HTTP", "callback_interval", 2)
    gen_resp = await MythicPayloadRPC(task).build_payload_from_MythicPayloadRPCResponse(orig)
    if gen_resp.status == MythicStatus.Success:
        # we know a payload is building, now we want it
        while True:
            resp = await MythicPayloadRPC(task).get_payload_by_uuid(gen_resp.uuid)
            if resp.status == MythicStatus.Success:
                if resp.build_phase == "success":
                    # it's done, so we can register a file for it
                    task.args.add_arg("template", resp.agent_file_id)
                    break
                elif resp.build_phase == "error":
                    raise Exception(
                        "Failed to build new payload: " + resp.error_message
                    )
                else:
                    await asyncio.sleep(1)
    return task
```

* `create_output` - Add a new response that the user can see.&#x20;
* `update_callback` - Update a piece of information about the current callback.
* `create_artifact` - Register a new artifact to be tracked within the current operation.
* `create_token` - Register Windows Token information on a host.&#x20;
* `delete_token` - Mark Windows Token as deleted
* `create_logon_session` - Register Windows Logon Session information on a host
* `delete_logon_session` - Mark Windows Logon Session as deleted
* `create_callback_token` - Register Windows Tokens to be associated with your callback. This allows you to then select those tokens when issuing tasking, assuming your Payload Type supports that level of tracking.
* `delete_callback_token` - Unregister a windows token with your callback
* `create_processes` - Register new processes on a host. This will feed into the unified callback listing in the user interface.
* `create_process` - Register a single new process
* `get_tasks` - Get the tasks and their associated token information for all tasks that haven't completed yet on a host.
* `create_keylog` - Register keystrokes with Mythic
* `create_credential` - Register new credentials with Mythic
* `search_database` - Specify a table and use regex matches to search for specific values in specific columns. This allows you to search for specific process names on a host for example. This currently only supports searching for process information but will be expanding going forward.
* `control_socks` - Ask Mythic to start a new socks instance/stop an existing instance for this callback on a specific port

{% hint style="info" %}
There can only be one socks instance per callback
{% endhint %}

## Example - Upload to Target

```python
from mythic_payloadtype_container.MythicCommandBase import *
import json
import base64
import sys
from mythic_payloadtype_container.MythicRPC import *

class UploadArguments(TaskArguments):
    def __init__(self, command_line):
        super().__init__(command_line)
        self.args = {
            "remote_path": CommandParameter(
                name="Remote Path",
                type=ParameterType.String,
                description="Path where the uploaded file will be written.",
                ui_position=2
            ),
            "file_id": CommandParameter(
                name="File to Upload",
                type=ParameterType.File,
                description="The file to be written to the remote path.",
                ui_position=1
            ),
            "overwrite": CommandParameter(
                 name="Overwrite Exiting File",
                 type=ParameterType.Boolean,
                 description="Overwrite file if it exists.",
                 default_value=False,
                 ui_position=3
             )
        }

    async def parse_arguments(self):
        self.load_args_from_json_string(self.command_line)


class UploadCommand(CommandBase):
    cmd = "upload"
    needs_admin = False
    help_cmd = "upload"
    description = "upload a file to the target."
    version = 1
    supported_ui_features = ["file_browser:upload"]
    author = "@xorrior"
    argument_class = UploadArguments
    attackmapping = []

    async def create_tasking(self, task: MythicTask) -> MythicTask:
        try:
            original_file_name = json.loads(task.original_params)["File to Upload"]
            if len(task.args.get_arg("remote_path")) == 0:
                task.args.add_arg("remote_path", original_file_name)
            elif task.args.get_arg("remote_path")[-1] == "/":
                task.args.add_arg("remote_path", task.args.get_arg("remote_path") + original_file_name)
            file_resp = await MythicRPC().execute("create_file", task_id=task.id,
                file=base64.b64encode(task.args.get_arg("file_id")).decode(),
                saved_file_name=original_file_name,
                delete_after_fetch=False,
            )
            if file_resp.status == MythicStatus.Success:
                task.args.add_arg("file_id", file_resp.response["agent_file_id"])
                task.display_params = f"{original_file_name} to {task.args.get_arg('remote_path')}"
            else:
                raise Exception("Error from Mythic: " + str(file_resp.error))
        except Exception as e:
            raise Exception("Error from Mythic: " + str(sys.exc_info()[-1].tb_lineno) + str(e))
        return task

```

The above example takes in a file from the operator and wants to upload it to a specific remote path on the target. In the `UploadArguments` class, we define two main arguments in `self.args`, one for the file and one for the remote path. By default, all arguments are `required`. In the `parse_arguments` function we expect to get a JSON string of our file and arguments, so we parse it as so. If it's not actually JSON, that function will throw an error and it'll be echoed back to the operator.

When it comes to the `create_tasking` function, we want to register this file in the Mythic database so we can track that the file exists and is being uploaded to a specific remote target. This helps us track artifacts. Taking this exact case an example, when you upload files, 2 things happen:

1. the `task.original_params` (this is what the user sees initially), we'll have the file swapped out with the name of the file they uploaded. So, if we uploaded a file called "bug.jpg", then an example of the `task.original_params` would be: `{"file": "bug.jpg", "remote_path": "/blah/evil.jpg"}`.&#x20;
2. The actual parameters that get sent to this function have the raw file contents though, so if you look at `task.args.get_arg("file")`, you'll get the raw bytes of the `bug.jpg` file. This is meant simply as a way to capture the name of the file as well as preventing files from cluttering up the UI for operators. When we send information down to Mythic to track the file, we need to send the base64 encoded contents of the file.

When we register the file, we want to track the original name of the file as well, this is why we load the original parameters and get the "file" value (which is the name of the file).

Once the file registration is done, we want the agent to be able to pull this down in chunks, so we swap out the raw contents of the file with a file UUID instead (remember, if you do `task.args.add_arg` for an argument that already exists, it just replaces the value).

