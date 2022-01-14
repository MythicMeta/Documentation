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

When an operator types a command in the UI, whatever the operator types (or whatever is populated based on the popup modal) gets sent to this function after the input is parsed and validated by the TaskArguments and CommandParameters functions mentioned in [Commands](commands.md).

It's here that the operator has full control of the task before it gets sent down to an agent. The task is currently in the "preprocessing" stage when this function is executed and allows you to do many things via Remote Procedure Calls (RPC) back to the Mythic server.

### Available Context

So, from this create\_tasking function, what information do you immediately have available?

* `task.task_id` - the task number associated with this task (not the UUID the agent sees, the integer value that operators see)
* `task.agent_task_id` - the UUID associated with this task that an agent sees
* `task.original_params` - the string of original parameters that was passed to this function before anything was swapped out for default values or edited.
* `task.completed` - boolean indicating if the task is marked as completed or not
* `task.operator` - the name of the operator that issued the task
* `task.status` - the current status of the task (defaults to MythicStatus.Success).
  * Options are `MythicStatus.Error, MythicStatus.Completed`, `MythicStatus.Processed`, `MythicStatus.Processing`
  * If you set the status to `Success`, then Mythic will set the task to `Submitted` so that it's ready to be picked up by the agent. If you set it to error, completed, processed, or processing, then the agent _won't_ pick it up.
* `task.callback` - information about the task's associated callback. This has a LOT of information, so let's break it down more:
  * `task.callback.build_parameters` - this is a dictionary of all the build parameters used to generate the payload that the callback is based on. This is the same sort of information from [Payload Type Info](payload-type-info.md#build-parameters). So, if you had a build parameter called "version", you could see that associated value with `task.callback.build_parameters.['version']`
  * `task.callback.c2info` - this is an array of dictionaries of the different c2 profiles built into the callback. Just like [Payload Type Info](payload-type-info.md#building) where you iterate over them, you can do the same here. For example, to see a parameter called "callback\_host" in our first c2 profile is `task.callback.c2info[0]['callback_host']`.
  * `task.callback.user` is the user for the callback
  * `task.callback.host` is the hostname of the callback
  * ... These match up with all the fields used for doing the initial callback - [Action: Checkin](../c2-related-development/c2-profile-code/agent-side-coding/initial-checkin.md#plaintext-checkin)
    * Something to note about `integrity_level` values - these are based on Windows standard integrity levels where 2 == normal account, 3 == high integrity (or `root`), and 4 == SYSTEM (but in most cases for operators, 3 and 4 are the same).
* `task.args` - access to the associated arguments class for this command that already has all of the values populated and validated. Let's say you have an argument called "remote\_path", you can access it via `task.args.get_arg("remote_path")` .
  * Want to change the value of that to something else? `task.args.add_arg("remote_path", "new value")`.
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
response.status == MythicRPCStatus.Success
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
* `create_output` - Add a new response that the user can see.
* `update_callback` - Update a piece of information about the current callback.
* `create_artifact` - Register a new artifact to be tracked within the current operation.
* `create_token` - Register Windows Token information on a host.
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
from mythic_payloadtype_container.MythicRPC import *
import json
import sys
import base64

class UploadArguments(TaskArguments):
    def __init__(self, command_line, **kwargs):
        super().__init__(command_line, **kwargs)
        self.args = [
            CommandParameter(
                name="file", cli_name="new-file", display_name="File to upload", type=ParameterType.File, description="Select new file to upload",
                parameter_group_info=[
                    ParameterGroupInfo(
                        required=True,
                        group_name="Default"
                    )
                ]
            ),
            CommandParameter(
                name="filename", cli_name="registered-filename", display_name="Filename within Mythic", description="Supply existing filename in Mythic to upload",
                type=ParameterType.ChooseOne,
                dynamic_query_function=self.get_files,
                parameter_group_info=[
                    ParameterGroupInfo(
                        required=True,
                        group_name="specify already uploaded file by name"
                    )
                ]
            ),
            CommandParameter(
                name="remote_path",
                cli_name="remote_path",
                display_name="Upload path (with filename)",
                type=ParameterType.String,
                description="Provide the path where the file will go (include new filename as well)",
                parameter_group_info=[
                    ParameterGroupInfo(
                        required=True,
                        group_name="Default",
                        ui_position=1
                    ),
                    ParameterGroupInfo(
                        required=True,
                        group_name="specify already uploaded file by name",
                        ui_position=1
                    )
                ]
            ),
        ]

    async def parse_arguments(self):
        if len(self.command_line) == 0:
            raise ValueError("Must supply arguments")
        raise ValueError("Must supply named arguments or use the modal")

    async def parse_dictionary(self, dictionary_arguments):
        self.load_args_from_dictionary(dictionary_arguments)

    async def get_files(self, callback: dict) -> [str]:
        file_resp = await MythicRPC().execute("get_file", callback_id=callback["id"],
                                              limit_by_callback=False,
                                              get_contents=False,
                                              filename="",
                                              max_results=-1)
        if file_resp.status == MythicRPCStatus.Success:
            file_names = []
            for f in file_resp.response:
                if f["filename"] not in file_names:
                    file_names.append(f["filename"])
            return file_names
        else:
            return []


class UploadCommand(CommandBase):
    cmd = "upload"
    needs_admin = False
    help_cmd = "upload"
    description = (
        "Upload a file to the target machine by selecting a file from your computer. "
    )
    version = 1
    supported_ui_features = ["file_browser:upload"]
    author = "@its_a_feature_"
    attackmapping = ["T1132", "T1030", "T1105"]
    argument_class = UploadArguments
    attributes = CommandAttributes(
        suggested_command=True
    )

    async def create_tasking(self, task: MythicTask) -> MythicTask:
        try:
            groupName = task.args.get_parameter_group_name()
            if groupName == "Default":
                file_resp = await MythicRPC().execute("get_file",
                                                    file_id=task.args.get_arg("file"),
                                                    task_id=task.id,
                                                    get_contents=False)
                if file_resp.status == MythicRPCStatus.Success:
                    original_file_name = file_resp.response[0]["filename"]
                    if len(task.args.get_arg("remote_path")) == 0:
                        task.args.add_arg("remote_path", original_file_name)
                    elif task.args.get_arg("remote_path")[-1] == "/":
                        task.args.add_arg("remote_path", task.args.get_arg("remote_path") + original_file_name)
                    task.display_params = f"{original_file_name} to {task.args.get_arg('remote_path')}"
                else:
                    raise Exception("Error from Mythic trying to get file: " + str(file_resp.error))
            elif groupName == "specify already uploaded file by name":
                # we're trying to find an already existing file and use that
                file_resp = await MythicRPC().execute("get_file", task_id=task.id,
                                                      filename=task.args.get_arg("filename"),
                                                      limit_by_callback=False,
                                                      get_contents=False)
                if file_resp.status == MythicRPCStatus.Success:
                    if len(file_resp.response) > 0:
                        task.args.add_arg("file", file_resp.response[0]["agent_file_id"])
                        task.args.remove_arg("filename")
                        task.display_params = f"existing {file_resp.response[0]['filename']} to {task.args.get_arg('remote_path')}"
                    elif len(file_resp.response) == 0:
                        raise Exception("Failed to find the named file. Have you uploaded it before? Did it get deleted?")
                else:
                    raise Exception("Error from Mythic trying to search files:\n" + str(file_resp.error))
        except Exception as e:
            raise Exception("Error from Mythic: " + str(sys.exc_info()[-1].tb_lineno) + " : " + str(e))
        return task

    async def process_response(self, response: AgentResponse):
        pass

```

The above example takes in a file from the operator and wants to upload it to a specific remote path on the target.

When it comes to the `create_tasking` function, there are a few potential things we could do. When a user supplies a file as part of tasking, the file is uploaded to Mythic ahead of your `create_tasking` function being executed. As part of that, you'll notice that when you specify `File` type parameters, what you'll actually get is a UUID instead of file contents. This makes it easier to work with files within Mythic instead of having to shuttle files unnecessarily between docker containers. We can still register new files with Mythic via the `create_file` RPC call, or we can update files via the `update_file` RPC call, or we can register new files with Mythic via the `create_file` RPC call. Taking this exact case an example, when you upload files, 2 things happen:

1. the `task.original_params` (this is what the user sees initially), we'll have the file swapped out with the UUID of the file they uploaded. So, if we uploaded a file called "bug.jpg", then an example of the `task.original_params` would be: `{"file": "UUID HERE", "remote_path": "/blah/evil.jpg"}`.
2. We want to present something meaningful to the user though when they look back at their tasking. They want to see that they uploaded `bug.jpg` to `/blah/evil.jpg`, not that they uploaded `random uuid here` to `/blah/evil.jpg`. So, we do an RPC call to get the information about the file that was uploaded and grab the `filename` attribute.

Once the file registration is done, we want the agent to be able to pull this down in chunks, so we keep the file UUID value (remember, if you do `task.args.add_arg` for an argument that already exists, it just replaces the value). Now, if you didn't want to chunk the contents of the file down to the agent and you just wanted the entire contents of the file as part of the parameters, you could swap out the `file` attribute with that value by doing something like `task.args.add_arg("file", file_resp.response[0]["contents"])` - assuming that when you did the original fetch for the file information that you set `get_contents=True`.
