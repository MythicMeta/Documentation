# Hooking Features

All of the following features describe information that can be included in responses. These sections describe some additional JSON formats and data that can be used to have your responses be tracked within Mythic or cause the creation of additional elements within Mythic (such as files, credentials, artifacts, etc).

You can hook multiple features in a single response because they're all unique.  For example, to display something to the user, it should be in the `user_output`field, such as:

```javascript
{
    "user_output": "Still working",
}

or even
{
    "user_output": "{\"key": \"nested json for user as string\"}"
}
```

### Reserved Keywords

When we talk about `Hooking Features` in the [Action: post\_response](../payload-type-development/create\_tasking/agent-side-coding/action-post\_response.md) message of an agent, we're really talking about a specific set of Dictionary key value pairs that have special meaning. All responses from the agent to the Mythic server already have to be in a structured format. Each of the following sections goes into what their reserved keywords mean, but some simpler ones are:

* task\_id - string - UUID associated with tasks
* user\_output - string - used with any command to display information back to the user
* completed - boolean - used with any command to indicate that the task is done (switches to the green completed icon)
* status - string - used to indicate that a command is not only done, but has encountered an error or some other status to return to the user
* process\_response - this is passed to your command's python file for processing in the `process_response` function.

## PayloadType Development Reference

As you're developing an agent to hook into these features, it's helpful to know where to look if you have questions. All of the Task, Command, and Parameter definitions/functions available to you are defined in the `mythic_container` PyPi package, which is hosted on the MythicMeta Organization on GitHub. Information about the Payload Type itself (BuildResponse, SupportedOS, BuildParameters, PayloadType, etc) can be found in the [PayloadBuilder.py](https://github.com/MythicMeta/Mythic\_PayloadType\_Container/blob/master/mythic\_payloadtype\_container/PayloadBuilder.py) file in the same PyPi repo.

## Message Keywords and Structure

Throughout this section, the payload type development section, and the c2 message format sections, you'll see a lot of information about message structure. Here is a quick "cheat sheet" reference guide with links to the appropriate sections for more information. The following is an example of a `get_tasking` request to Mythic with almost every possible field added:

```json
{ 
    "action": "get_tasking",
    "tasking_size": 1,
    "responses": [
        {
            "task_id": "uuid",
            "user_output": "something to show to the user",
            "completed": false,
            "status": "custom status here",
            "file_browser": {
                "host": "abc.com",
                "is_file": false,
                "permissions": {
                    "customField": customVal
                },
                "name": "C:\\",
                "parent_path": "",
                "success": true,
                "access_time": 1700164038000,
                "modify_time": 1700164038000,
                "size": 2300,
                "update_deleted": false,
                "files": [
                    "is_file": false,
                    "permissions": {
                        "customField": customVal
                    },
                    "name": "Users",
                    "access_time": 1700164038000,
                    "modify_time": 1700164038000,
                    "size": 12345
                ]
            },
            "removed_files": [
                {
                    "host": "abc.com", 
                    "path": "C:\\Users\\itsafeature\\Desktop\\evil.exe"
                }
            ],
            "credentials": [
                {
                    "credential_type": "plaintext",
                    "realm": "domain.com",
                    "account": "itsafeature",
                    "credential": "oh no my password!",
                    "comment": "scraped from lsass",
                    "metadata": "anything else you want to add"
                }
            ],
            "artifacts": [
                {
                    "base_artifact": "Process Create",
                    "artifact": "cmd.exe /C evil.exe",
                    "host": "abc.com"
                }
            ],
            "processes": [
                {
                    "host": "abc.com",
                    "process_id": 245,
                    "parent_process_id": 244,
                    "architecture": "x64",
                    "bin_path": "C:\\Users\\itsafeature\\Desktop\\evil.exe",
                    "name": "evil.exe",
                    "user": "itsafeature",
                    "command_line": "C:\\Users\\itsafeature\\Desktop\\evil.exe -f 2",
                    "integrity_level": 2,
                    "start_time": 1700164038000,
                    "description": "totally not malware: TM",
                    "signer": "",
                    "protected_process_level": 0,
                    "update_deleted": false,
                }
            ],
            "edges": [
                {
                    "source": "my uuid",
                    "destination": "uuid of remote callback",
                    "action": "remove",
                    "c2_profile": "smb",
                }
            ],
            "commands": [
                {
                    "action": "add",
                    "cmd": "shell"
                }
            ],
            "keylogs": [
                {
                    "window_title": "Notepad",
                    "user": "itsafeature",
                    "keystrokes": "password: abc123"
                }
            ],
            "tokens": [
                {
                    "action": "add",
                    "token_id": 34857,
                    "user": "acme\\bob",
                    "groups": "",
                    "privileges": "",
                    "thread_id": 12345,
                    "process_id": 2344,
                    "session_id": 1,
                    "logon_sid": "",
                    "integrity_level_sid": ""
                    "restricted": false,
                    "default_dacl": "",
                    "handle": 0,
                    "capabilities": "",
                    "app_container_sid": "",
                    "app_container_number": 0                    
                }
            ],
            "callback_tokens": [
                {
                    "action": "add",
                    "host": "abc.com",
                    "token_id": 34857,
                    "token": {
                        // same info from tokens if you wanted to add/update that data
                    }
                }
            ],
            "download": {
                "total_chunks": 4,
                "chunk_size": 512000,
                "host": "abc.com",
                "is_screenshot": false,
                "filename": "evil.exe",
                "full_path": "C:\\Users\\itsafeature\\Desktop\\evil.exe",
            },
            "upload": {
                "file_id": "uuid here",
                "host": "abc.com",
                "chunk_size": 512000,
                "chunk_num": 1,
                "full_path": "C:\\Users\\itsafeature\\Desktop\\replaced.exe"
            },
            "alerts": [{
                "alert": "lost connection to remote agent", 
                "level": "warning", 
                "source": "disconnection warning",
                "send_webhook": false,
            }],
            "process_response": {
                "custom field": "custom val"
            }
        }
    ],
    "alerts": [{
        "alert": "edr detected", 
        "level": "warning", 
        "source": "edr detection",
        "send_webhook": true,
        "webhook_alert": {
            "edr": "some edr name",
            "pid": 345
        }
    }],
    "edges": [{
        "action": "add", 
        "source": "my uuid", 
        "destination": "remote uuid",
        "c2_profile": "smb",
        "metadata": "anything else you want to add about the connection"
    }],
    "delegates": [{
        "c2_profile": "tcp",
        "message": "base64 message",
        "uuid": "some uuid tracker here"
    }],
    "socks": [{
        "server_id": 2345, 
        "data": "base64", 
        "exit": false
    }],
    "rpfwd": [{
        "server_id": 12345, 
        "data": "base64", 
        "exit": false
    }],
    "interactive": [{
        "task_id": "uuid of task that started interactive session", 
        "message_type": 0, 
        "data": "base64"
    }],
}
```

* [Delegates](../payload-type-development/create\_tasking/agent-side-coding/delegates.md)
* [Socks](socks.md)
* [Rpfwd](rpfwd.md)
* [Interactive](interactive-tasking.md)
* [Edges](linking-agents/action-p2p\_info.md)
* [Alerts](alerts.md)
* [Upload](action-upload.md)
* [Download](download.md)
* [Callback Tokens](tokens.md)
* [Tokens](tokens.md)
* [Keylogs](keylog.md)
* [ProcessResponse](../payload-type-development/process-response.md)
* [Commands](commands.md)
* [Processes](process\_list.md)
* [Artifacts](artifacts.md)
* [Credentials](credentials.md)
* RemovedFiles
* [FileBrowser](file-browser.md)
