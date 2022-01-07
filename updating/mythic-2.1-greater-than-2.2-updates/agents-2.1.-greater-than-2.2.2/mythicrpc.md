# MythicRPC

## MythicRPC in 2.2.7+

The RPC functionality within Mythic as of 2.2.7 is more dynamic, allowing more functionality to be added to the back-end and automatically usable by the Payload Type containers without requiring new PyPi packages or new Docker images. To facilitate this, you can always get the latest RPC available functionality within your tasking via:

```python
from mythic_payloadtype_container.MythicRPC import *
import sys

async def create_tasking(self, task: MythicTask) -> MythicTask:
        resp = await MythicRPC().get_functions()
        print(resp.response)
        sys.stdout.flush()
        return task
```

That will print out all of the information for the available functions if you're ever in doubt about what's available or how to call the functions. When you find a function you want to call, you do it as follows:

```python
file_resp = await MythicRPC().execute("create_file", task_id=task.id,
                file=base64.b64encode(task.args.get_arg("file_id")).decode(),
                saved_file_name=original_file_name,
                delete_after_fetch=False,
            )
```

where you always call `await MythicRPC().execute` with the first parameter being the name of the function to call and all of the other arguments being passed in like normal function arguments.

## 2.3.0+ Functions

The function set is moving towards a standard nomenclature - `create_*` for when you want to create/register/add something to the database, `get_*` for when you want to fetch something from the database, and `delete_*` when you want to remove something from the database or mark it as deleted. The current set of functionality for 2.2.8 is as follows:

```python
create_file(task_id: int, file: str, delete_after_fetch: bool = True, saved_file_name: str = None, is_screenshot: bool = False, is_download: bool = False, remote_path: str = None, host: str = None) -> dict
    Creates a FileMeta object in Mythic's database and writes contents to disk with a random UUID filename.
    This file can then be fetched via the returned file UUID.
    :param task_id: The ID number of the task performing this action (task.id)
    :param file: The base64 contents of the file to register
    :param delete_after_fetch: Should Mythic delete the file from disk after the agent fetches it. This also marks the file as deleted in the UI. This is useful if the file is a temporary file that doesn't necessarily need long-term tracking within Mythic.
    :param saved_file_name: The name of the file (if none supplied, a random UUID4 value will be used)
    :param is_screenshot: Is this file a screenshot reported by the agent? If so, this will cause it to show up in the screenshots page.
    :param is_download: Is this file the result of downloading something from the agent? If so, this will cause it to show up in the Files page under Downloads
    :param remote_path: Does this file exist on target? If so, provide the full remote path here
    :param host: If this file exists on a target host, indicate it here in conjunction with the remote_path argument
    :return: Dict of a FileMeta object
    Example: this takes two arguments - ParameterType.String for `remote_path` and ParameterType.File for `file`
        async def create_tasking(self, task: MythicTask) -> MythicTask:
            try:
                original_file_name = json.loads(task.original_params)["file"]
                if len(task.args.get_arg("remote_path")) == 0:
                    task.args.add_arg("remote_path", original_file_name)
                elif task.args.get_arg("remote_path")[-1] == "/":
                    task.args.add_arg("remote_path", task.args.get_arg("remote_path") + original_file_name)
                file_resp = await MythicRPC().execute("create_file", task_id=task.id,
                    file=base64.b64encode(task.args.get_arg("file")).decode(),
                    saved_file_name=original_file_name,
                    delete_after_fetch=False,
                )
                if file_resp.status == MythicStatus.Success:
                    task.args.add_arg("file", file_resp.response["agent_file_id"])
                    task.display_params = f"{original_file_name} to {task.args.get_arg('remote_path')}"
                else:
                    raise Exception("Error from Mythic: " + str(file_resp.error))
            except Exception as e:
                raise Exception("Error from Mythic: " + str(sys.exc_info()[-1].tb_lineno) + str(e))
            return task
```

```python
get_file(task_id: int = None, callback_id: int = None, filename: str = None, limit_by_callback: bool = True, max_results: int = 1, file_id: str = None, get_contents: bool = True) -> dict
    Get file data and contents by name (ex: from create_file and a specified saved_file_name parameter).
    The search can be limited to just this callback (or the entire operation) and return just the latest or some number of matching results.
    :param task_id: The ID number of the task performing this action (task.id) - if this isn't provided, the callback id must be provided
    :param callback_id: The ID number of the callback for this action - if this isn't provided, the task_id must be provided
    :param filename: The name of the file to search for (Case sensitive)
    :param file_id: If no filename specified, then can search for a specific file by this UUID
    :param limit_by_callback: Set this to True if you only want to search for files that are tied to this callback. This is useful if you're doing this as part of another command that previously loaded files into this callback's memory.
    :param max_results: The number of results you want back. 1 will be the latest file uploaded with that name, -1 will be all results.
    :param get_contents: Boolean of if you want to fetch file contents or just metadata
    :return: An array of dictionaries representing the FileMeta objects of all matching files. When "get_contents" is True, each entry in this array will also have a "contents" key with the base64 representation of the associated file if it hasn't been deleted, or None if it has.
    For an example-
    resp = await MythicRPC().execute("get_file", task_id=task.id, filename="myAssembly.exe")
    resp.response <--- this is an array
    resp.response[0] <--- this is the most recently registered matching file where filename="myAssembly.exe"
    resp.response[0]["filename"] <-- the filename of that first result
    resp.response[0]["contents"] <--- the base64 representation of that file
    All of the possible dictionary keys are available at https://github.com/its-a-feature/Mythic/blob/master/mythic-docker/app/database_models/model.py for the FileMeta class
```

```python
get_payload(payload_uuid: str, get_contents: bool = True) -> dict
    Get information about a payload and its contents
    :param payload_uuid: The UUID for the payload you're interested in
    :param get_contents: Whether or not you want to fetch the contents of the file or just the metadata
    :return: dictionary representation of the Payload object
    Example:
        async def create_tasking(self, task: MythicTask) -> MythicTask:
            try:
                gen_resp = await MythicRPC().execute("create_payload_from_uuid", task_id=task.id,
                                                     payload_uuid=task.args.get_arg("template"))
                if gen_resp.status == MythicStatus.Success:
                    # we know a payload is building, now we want it
                    while True:
                        resp = await MythicRPC().execute("get_payload", payload_uuid=gen_resp.response["uuid"])
                        if resp.status == MythicStatus.Success:
                            if resp.response["build_phase"] == "success":
                                task.args.add_arg("template", resp.response["file"]["agent_file_id"])
                                task.display_params = f"new Apfell payload ({resp.response['uuid']}) with description {resp.response['tag']}"
                                break
                            elif resp.response["build_phase"] == "error":
                                raise Exception(
                                    "Failed to build new payload: " + str(resp.error)
                                )
                            else:
                                await asyncio.sleep(1)
                        if resp.status == MythicStatus.Error:
                            raise Exception("Failed to get information about new payload:\n" + resp.error)
                else:
                    raise Exception("Failed to generate new payload:\n" + gen_resp.error)
            except Exception as e:
                raise Exception("Error trying to call RPC:\n" + str(e))
            return task
```

```
search_payloads(callback_id: int, payload_types: [str] = None, include_auto_generated: bool = False, description: str = "",
                          filename: str = "", build_parameters: dict = None) -> dict:
    """
    Search payloads based on payload type, if it was auto generated, the description, the filename, or build parameter values.
    Note: This does not search payloads that have been deleted.
    :param callback_id: The ID of the callback this search is for, this is what's used to limit your search to the right operation.
    :param payload_types: The names of the associated payload type if you want to restrict results
    :param include_auto_generated: Boolean if you want to include payloads that were automatically generated as part of tasking
    :param description: If you want to search for payloads with certain information in their description, this functions like an igrep search
    :param filename: If you want to search for payloads with certain filenames, this functions like an igrep search
    :param build_parameters: If you want to limit your search based on certain build parameters (maybe shellcode for example),
        then you can specify this dictionary of {"agent name": {"build_param_name": "build_param_value"}}
    :return: An array of dictionaries where each entry is one matching payload. Each dictionary entry contains the following:
        uuid - string
        description -string
        operator - string
        creation_time - string
        payload_type - string
        operation - string
        wrapped_payload - boolean (true if this payload wraps another payload)
        deleted - boolean
        build_container - string
        build_phase - string
        build_message - string
        build_stderr - string
        build_stdout - string
        callback_alert - boolean (true if this payload will attempt to hit the operation's webhook when a new callback is generated)
        auto_generated - boolean (true if this payload is auto generated by a task)
        task - dictionary of information about the associated task
        file - dictionary of information about the associated file
        os - string
    """
```

```python
encrypt_message(message: dict, target_uuid: str, c2_profile_name: str):
    Given a dictionary agent message, submit it to Mythic to encrypt with a target callback/payload's encryption keys
    :param message: the dictionary message
    :param target_uuid: the UUID of the payload/stager/callback that will receive the encrypted message
    :param c2_profile_name: the name of the c2 profile that this message will be sent over
    :return: The final base64 and encrypted message
```

```python
decrypt_message(message: str, c2_profile_name: str):
    Given an encrypted message from an agent, decrypt it based on the C2 profile that received it
    :param message: The base64 of the message from an agent
    :param c2_profile_name: the name of the c2 profile where this message came from
    :return: the dictionary representation of the message for Mythic
```

```python
get_commands(callback_id: int = None, loaded_only: bool = False,
                       payload_type_name: str = None, commands: [str] = None, os: str = None):
    Get an array of dictionaries of all the possible commands for the specified callback or payload type
    :param callback_id: the id of the callback in question
    :param loaded_only: specify this as True to only include commands currently loaded into this callback
    :param payload_type_name: specify this to fetch all possible commands for a specific payload type
    :param commands: specify an array of command names along with the payload_type_name to fetch information
        about only the listed commands for the specified payload type
    :param os: Specify the OS that's associated with the payload_type_name so that commands can be filtered
    :return: an array of dictionaries representing all of the requested commands for that payload type.
    When returning all possible commands for this callback, commands are still filtered by their supported_os attributes
```

```python
add_commands_to_payload(payload_uuid: str, commands: [str]):
    Register additional commands that are in the payload. This is useful if a user selects command X to include in a payload, but command X needs command Y.
    A common example would be if command X is a script_only command or will end up delegating additional commands.
    :param payload_uuid: The UUID of the payload that you're adding commands to.
    :param commands: An array of command names that should be added to this payload.
    :return: Success or Error
```

```
get_tasks(task_id: int, host: str = None) -> dict
    Get all of the currently running tasks on the current host or on a specific host
    :param task_id: The ID number of the task performing this action (task.id)
    :param host: The name of the host to check for running tasks
    :return: An array of dictionaries representing the tasks running
```

```python
get_responses(task_id: int) -> dict
    For a given Task, get all of the user_output, artifacts, files, and credentials that task as created within Mythic
    :param task_id: The TaskID you're interested in (i.e. task.id)
    :return: A dictionary of the following format:
    {
      "user_output": array of dictionaries where each dictionary is user_output message for the task,
      "artifacts": array of dictionaries where each dictionary is an artifact created for the task,
      "files": array of dictionaries where each dictionary is a file registered as part of the task,
      "credentials": array of dictionaries where each dictionary is a credential created as part of the task.
    }
```

```python
create_payload_from_uuid(task_id: int, payload_uuid: str, generate_new_random_values: bool = True, new_description: str = None, remote_host: str = None, filename: str = None) -> dict
    Given an existing Payload UUID, generate a new copy with a potentially new description, new filename, new random values, and specify that it'll exist on a certain host. This is useful for spawn or lateral movement tasks where you want to potentially change up IOCs and provide new, more informative, descriptions for callbacks.
    :param task_id: The ID number of the task performing this action (task.id)
    :param payload_uuid: The UUID of the payload we're interested in
    :param generate_new_random_values: Set this to True to generate new random values for C2 Profile parameters that are flagged as randomized
    :param new_description: Provide a custom new description for the payload and callbacks associated from it. If you don't provide one, a generic one will be generated
    :param remote_host: Indicate the hostname of the host this new payload is deployed to. If one isn't specified, you won't be able to link to it without first telling Mythic that this payload exists on a certain host via the Popup Modals.
    :param filename: New filename for the payload. If one isn't supplied, a random UUID will be generated
    :return: dictionary representation of the payload that was created
    Example:
        async def create_tasking(self, task: MythicTask) -> MythicTask:
            try:
                gen_resp = await MythicRPC().execute("create_payload_from_uuid", task_id=task.id,
                                                     payload_uuid=task.args.get_arg("template"))
                if gen_resp.status == MythicStatus.Success:
                    # we know a payload is building, now we want it
                    while True:
                        resp = await MythicRPC().execute("get_payload", payload_uuid=gen_resp.response["uuid"])
                        if resp.status == MythicStatus.Success:
                            if resp.response["build_phase"] == "success":
                                task.args.add_arg("template", resp.response["file"]["agent_file_id"])
                                task.display_params = f"new Apfell payload ({resp.response['uuid']}) with description {resp.response['tag']}"
                                break
                            elif resp.response["build_phase"] == "error":
                                raise Exception(
                                    "Failed to build new payload: " + str(resp.error)
                                )
                            else:
                                await asyncio.sleep(1)
                        if resp.status == MythicStatus.Error:
                            raise Exception("Failed to get information about new payload:\n" + resp.error)
                else:
                    raise Exception("Failed to generate new payload:\n" + gen_resp.error)
            except Exception as e:
                raise Exception("Error trying to call RPC:\n" + str(e))
            return task
```

```
create_processes(task_id: int, processes: dict) -> dict
    Create processes in bulk. The parameters in the "processes" dictionary are the same as those in the `create_process` RPC call.
    :param task_id: The ID number of the task performing this action (task.id)
    :param processes: Dictionary of the processes you want to create - the key value pairs are the same as the parameters to the `create_process` RPC call.
    :return: Success or Error (nothing in the `response` attribute)
```

```
create_process(task_id: int, host: str, process_id: int, parent_process_id: int = None, architecture: str = None, name: str = None, bin_path: str = None, user: str = None, command_line: str = None, integrity_level: int = None, start_time: str = None, description: str = None, signer: str = None) -> dict
    Create a new process within Mythic.
    :param task_id: The ID number of the task performing this action (task.id)
    :param host: The host where this process exists
    :param process_id: The process ID
    :param parent_process_id: The process's parent process ID
    :param architecture: The architecture for the process (x86, x64, arm, etc)
    :param name: The name of the process
    :param bin_path: The path to the binary that's executed
    :param user: The user context that the process is executing
    :param command_line: The command line that's spawned with the process
    :param integrity_level: The integrity level of the process
    :param start_time: When the process started
    :param description: The description of the process
    :param signer: The process' signing information
    :return: Success or Error (nothing in the `response` attribute)
```

```
create_artifact(task_id: int, artifact_type: str, artifact: str, host: str = None) -> dict
    Create a new artifact for a certain task on a host
    :param task_id: The ID number of the task performing this action (task.id)
    :param artifact_type: What kind of artifact is this (Process Create, File Write, etc). If the type specified doesn't exist, it will be created
    :param artifact: The actual artifact that was created
    :param host: Which host the artifact was created on. If none is provided, the current task's host is used
    :return: Success or error (nothing in the `response` attribute)
```

```
create_keylog(task_id: int, keystrokes: str, user: str = None, window_title: str = None) -> dict
    Create a new keylog entry in Mythic.
    :param task_id: The ID number of the task performing this action (task.id)
    :param keystrokes: The keys that are being registered
    :param user: The user that performed the keystrokes. If you don't supply this, "UNKNOWN" will be used.
    :param window_title: The title of the window where the keystrokes came from. If you don't supply this, "UNKNOWN" will be used.
    :return: Success or Error (nothing in the `response` attribute)
```

```
create_output(task_id: int, output: str) -> dict
    Add a message to the output for a task that the operator can see
    :param task_id: The ID number of the task performing this action (task.id)
    :param output: The message you want to send.
    :return: Status of if you successfully posted or not (nothing in the `response` attribute)
    Example:
        async def create_tasking(self, task: MythicTask) -> MythicTask:
            resp = await MythicRPC().execute("create_output", task_id=task.id, output="hello")
            if resp.status != MythicStatus.Success:
                task.status = MythicStatus.Error
                raise Exception(resp.error)
            return task
```

```
create_event_message(task_id: int, message: str, warning: bool = False) -> dict
    Create a message in the Event feed within the UI as an info message or as a warning
    :param task_id: The ID number of the task performing this action (task.id)
    :param message: The message you want to send
    :param warning: If this is True, the message will be a "warning" message
    :return: success or error (nothing in the `response` attribute)
```

```
create_credential(task_id: int, credential_type: str, account: str, realm: str, credential: str, metadata: str = '', comment: str = None) -> dict
    Create a new credential within Mythic to be leveraged in future tasks
    :param task_id: The ID number of the task performing this action (task.id)
    :param credential_type: The type of credential we're storing (plaintext, hash, ticket, certificate, token)
    :param account: The account associated with the credential
    :param realm: The realm for the credential (sometimes called the domain)
    :param credential: The credential value itself
    :param metadata: Any additional metadata you want to store about the credential
    :param comment: Any comment you want to store about it the credential
    :return: Success or Error (nothing in the `response` attribute)
```

```
create_file_browser(task_id: int, host: str, name: str, full_path: str, permissions: dict = None, access_time: str = '', modify_time: str = '', comment: str = '', is_file: bool = True, size: str = '', success: bool = True, files: [<class 'dict'>] = None, update_deleted: bool = False) -> dict
    Add file browser content to the file browser user interface.
    :param task_id: The ID number of the task performing this action (task.id)
    :param host: Which host this data is from (useful for remote file listings)
    :param name: Name of the file/folder that was listed
    :param full_path: Full path of the file/folder that was listed (useful in case the operator said to ls `.` or a relative path)
    :param permissions: Dictionary of permissions. The key/values here are completely up to you and are displayed as key/value pairs in the UI
    :param access_time: String representation of when the file/folder was last accessed
    :param modify_time: String representation of when the file/folder was last modified
    :param comment: Any comment you might want to add to this file/folder
    :param is_file: Is this a file?
    :param size: Size of the file (can be an int or something human readable, like 10MB)
    :param success: True/False if you successfully listed this file. A False value (like from an access denied) will appear as a red X in the UI
    :param files: Array of dictionaries of information for all of the files in this folder (or an empty array of this is a file). Each dictionary has all of the same pieces of information as the main folder itself.
    :param update_deleted: True or False indicating if this file browser data should be used to automatically update deleted files for the listed folder. This defaults to false, but if set to true and there are files that Mythic knows about for this folder that the passed-in data doesn't include, it will be marked as deleted.
    :return: success or error (nothing in the `response` attribute)
```

```
create_payload_on_host(task_id: int, payload_uuid: str, host: str) -> dict
    Register within Mythic that the specified payload exists on the specified host as a result of this tasking
    :param task_id: The ID number of the task performing this action (task.id)
    :param payload_uuid: The payload that will be associated with the host
    :param host: The host that will have the payload on it
    :return: success or error (nothing in the `response` attribute)
```

```
create_logon_session(task_id: int, LogonId: int, host: str = None, **kwargs) -> dict
    Create a new logon session for this host
    :param task_id: The ID number of the task performing this action (task.id)
    :param LogonId: The integer logon identifier value that uniquely identifies this logon session on this host
    :param host: The host where this logon session exists
    :param kwargs: The `Mythic/mythic-docker/app/database_models/model.py` LogonSession class has all of the possible values you can set when creating/updating logon sessions. There are too many to list here individually, so a generic kwargs is specified.
    :return: Success or Error (nothing in the `response` attribute)
```

```
create_callback_token(task_id: int, TokenId: int, host: str = None) -> dict
    Associate a token with a callback for usage in further tasking.
    :param task_id: The ID number of the task performing this action (task.id)
    :param TokenId: The token you want to associate with this callback
    :param host: The host where the token exists
    :return: Success or Error (nothing in the `response` attribute)
```

```
create_token(task_id: int, TokenId: int, host: str = None, **kwargs) -> dict
    Create or update a token on a host. The `TokenId` is a unique identifier for the token on the host and is how Mythic identifies tokens as well. A token's `AuthenticationId` is used to link a Token to a LogonSession per Windows documentation, so when setting that value, if the associated LogonSession object doesnt' exist, Mythic will make it.
    :param task_id: The ID number of the task performing this action (task.id)
    :param TokenId: The integer token identifier value that uniquely identifies this token on this host
    :param host: The host where the token exists
    :param kwargs: The `Mythic/mythic-docker/app/database_models/model.py` Token class has all of the possible values you can set when creating/updating tokens. There are too many to list here individually, so a generic kwargs is specified.
    :return: Dictionary representation of the token created
```

```
delete_token(TokenId: int, host: str) -> dict
    Mark a specific token as "deleted" on a specific host.
    :param TokenId: The token that should be deleted
    :param host: The host where this token exists
    :return: success or error (nothing in the `response` attribute)
```

```
create_agentstorage(unique_id: str, data: bytes):
    Allow Payload Types and Translation containers to store arbitrary data within the database that doesn't fit
        somewhere else in Mythic's current schema
    :param unique_id: A unique string identifier
    :param data:
    :return: {"unique_id": "unique id here", "data": "base64 of data here"}
```

```
get_agentstorage(unique_id: str):
    Allow Payload Types and Translation containers to fetch arbitrary data within the database that doesn't fit
        somewhere else in Mythic's current schema
    :param unique_id: A unique string identifier
    :return: {"unique_id": "unique id here", "data": "base64 of data here"}
```

```
delete_agentstorage(unique_id: str):
    Allow Payload Types and Translation containers to delete arbitrary data within the database that doesn't fit
        somewhere else in Mythic's current schema
    :param unique_id: A unique string identifier
    :return: Success or Error
```

```
delete_file_browser(task_id: int, file_path: str, host: str = None) -> dict
    Mark a file in the file browser as deleted (typically as part of a manual removal via a task)
    :param task_id: The ID number of the task performing this action (task.id)
    :param file_path: The full path to the file that's being removed
    :param host: The host where the file existed. If you don't specify a host, the callback's host is used
    :return: Success or Error (nothing in the `response` attribute)
```

```
delete_logon_session(LogonId: int, host: str) -> dict
    Mark a specified logon session as "deleted" on a specific host
    :param LogonId: The Logon Session that should be deleted
    :param host: The host where the logon session used to be
    :return: Success or Error (nothing in the `response` attribute)
```

```
delete_callback_token(task_id: int, TokenId: int, host: str = None) -> dict
    Mark a callback token as no longer being associated
    :param task_id: The ID number of the task performing this action (task.id)
    :param TokenId: The Token you want to disassociate from the task's callback
    :param host: The host where the token exists
    :return: Success or Error (nothing in the `response` attribute)
```

```
update_callback(task_id: int, user: str = None, host: str = None, pid: int = None, ip: str = None, external_ip: str = None, description: str = None, integrity_level: int = None, os: str = None, architecture: str = None, domain: str = None, extra_info: str = None, sleep_info: str = None) -> dict
    Update this task's associated callback data.
    :param task_id: The ID number of the task performing this action (task.id)
    :param user: The new username
    :param host: The new hostname
    :param pid: The new process identifier
    :param ip: The new IP address
    :param external_ip: The new external IP address
    :param description: The new description
    :param integrity_level: The new integrity level
    :param os: The new operating system information
    :param architecture: The new architecture
    :param domain: The new domain
    :param extra_info: The new "extra info" you want to store
    :param sleep_info: The new sleep information for the callback
    :return: Success or error (nothing in the `response` attribute)
```

```
update_task_status(task_id: int, status: str, completed: bool = None):
    Update a task's status to a custom value and optionally mark a task as completed
    :param task_id: The task you want to update (i.e. task.id in you create_tasking)
    :param status: The string value of the status you want to set
    :param completed: Optional boolean value to mark the task as completed
    :return: Status indicating success or error on if the task was updated or not.
```

```
search_database(table: str, task_id: int = None, callback_id: int = None, **kwargs) -> dict
    Search the Mythic database for some data. Data is searched by regular expression for the fields specified. Because the available fields depends on the table you're searching, that argument is a generic python "kwargs" value.
    :param task_id: The ID number of the task performing this action (task.id) - if this isn't supplied, callback_id must be supplied
    :param callback_id: The ID number of the callback performing this action - if this isn't supplied, task_id must be supplied
    :param table: The name of the table you want to query. Currently only options are: process, token, file_browser. To search files (uploads/downloads/hosted), use `get_file`
    :param kwargs: These are the key=value pairs for how you're going to search the table specified. For example, searching processes where the name of "bob" and host that starts with "spooky" would have kwargs of: name="bob", host="spooky*"
    :return: an array of dictionaries that represent your search. If your search had no results, you'll get back an empty array
```

```
control_socks(task_id: int, port: int, start: bool = False, stop: bool = False) -> dict
    Start or stop SOCKS 5 on a specific port for this task's callback
    :param task_id: The ID number of the task performing this action (task.id)
    :param port: The port to open for SOCKS 5
    :param start: Boolean for if SOCKS should start
    :param stop: Boolean for if SOCKS should stop
    :return: Status message of if it completed successfully (nothing in the `response` attribute)
    Example:
        async def create_tasking(self, task: MythicTask) -> MythicTask:
            if task.args.get_arg("action") == "start":
                resp = await MythicRPC().execute("control_socks", task_id=task.id, start=True, port=task.args.get_arg("port"))
                if resp.status != MythicStatus.Success:
                    task.status = MythicStatus.Error
                    raise Exception(resp.error)
            else:
                resp = await MythicRPC().execute("control_socks", task_id=task.id, stop=True, port=task.args.get_arg("port"))
                if resp.status != MythicStatus.Success:
                    task.status = MythicStatus.Error
                    raise Exception(resp.error)
            return task
```

```
update_loaded_commands(task_id: int, commands: [str], add: bool = None, remove: bool = None):
    Add or Remove loaded commands for the callback associated with task_id
    :param task_id: The task doing the modifications
    :param commands: The list of command names to add/remove
    :param add: Boolean set to True if you want to add the commands to the callback associated with task_id
    :param remove: Boolean set to True if you want to remove teh commands from the callback assocaited with task_id
    :return: Status for success or error   
```

```python
create_subtask(parent_task_id: int, command: str, params: str = '', files: dict = None, subtask_callback_function: str = None, subtask_group_name: str = None, tags: [<class 'str'>] = None, group_callback_function: str = None) -> dict
    Issue a new task to the current callback as a child of the current task.
    You can use the "subtask_callback_function" to provide the name of the function you want to call when this new task enters a "completed=True" state.
    If you issue create_subtask_group, the group name and group callback functions are propagated here
    :param parent_task_id: The id of the current task (task.id)
    :param command: The name of the command you want to use
    :param params: The parameters you want to issue to that command
    :param files: If you want to pass along a file to the task, provide it here (example provided)
    :param subtask_callback_function: The name of the function to call on the _parent_ task when this function exits
    :param subtask_group_name: An optional name of a group so that tasks can share a single callback function
    :param tags: A list of strings of tags you want to apply to this new task
    :param group_callback_function: If you're grouping tasks together, this is the name of the shared callback function for when they're all in a "completed=True" state
    :return: Information about the task you just created
    
create_subtask_group(parent_task_id: int, tasks: [<class 'dict'>], subtask_group_name: str = None, tags: [<class 'str'>] = None, group_callback_function: str = None) -> dict
    Create a group of subtasks at once and register a single callback function when the entire group is done executing.
    :param parent_task_id: The id of the parent task (i.e. task.id)
    :param tasks: An array of dictionaries representing the tasks to create. An example is shown below.
    :param subtask_group_name: The name for the group. If one isn't provided, a random UUID will be used instead
    :param tags: An optional list of tags to apply to all of the subtasks created.
    :param group_callback_function: The name of the function to call in the _parent_ task when all of these subtasks are done.
    :return: An array of dictionaries representing information about all of the subtasks created.
```

## Calling C2 RPC Functions

Your Payload Type container can actually call RPC functions defined within your C2 profile as well. You must have Mythic 2.2.8 and a PayloadType container version of at least 9 to leverage this functionality. These just have a slightly different format:

```python
resp = await MythicRPC().execute_c2rpc(c2_profile="http", function_name="test", task_id=task.id,
        message="hi")
```

The key things to notice here are:

1. the function you execute is called `execute_c2rpc` instead of just `execute`
2. The `message` parameter is always a string. If you need to send a dictionary to your C2 profile, use `json.dumps({dictionary here})` (make sure you `import json` at the top)
3. Your response back will have the same `status` , `response`, and `error` as the other kind of RPC functions.

### Defining your own C2 RPC Functions

If you are creating your own C2, you can create your own C2 RPC functions! Inside of your `mythic/c2_functions` folder for your C2 Profile, create a file called `C2_RPC_functions.py` (it might already exist for you). This is where you can create as many RPC function endpoints as you want! They just have the following format:

```python
from mythic_c2_container.C2ProfileBase import *
import sys

# request is a dictionary: {"action": func_name, "message": "the input",  "task_id": task id num}
# must return an RPCResponse() object and set .status to an instance of RPCStatus and response to str of message
async def test(request):
    response = RPCResponse()
    response.status = RPCStatus.Success
    response.response = "hello"
    return response
```
