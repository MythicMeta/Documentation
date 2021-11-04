# Hooking Features

All of the following features describe information that can be included in responses. These sections describe some additional JSON formats and data that can be used to have your responses be tracked within Mythic or cause the creation of additional elements within Mythic (such as files, credentials, artifacts, etc).

You can hook multiple features in a single response because they're all unique. To display something to the user, it should be in the `user_output`field, such as:

```javascript
{
    "user_output": "Still working",
}

or even
{
    "user_output": "{\"key": \"nested json for user as string\"}"
}
```

The various styles of output are described in the follow-on pages:

* [Download](download.md)
* [Keylog](keylog.md)
* [Process\_List](process\_list.md)
* [Artifacts](artifacts.md)
* [Credentials](credentials.md)
* [P2P Connections](action-p2p\_info.md)

### Reserved Keywords

When we talk about `Hooking Features` in the [Action: post\_response](../c2-related-development/c2-profile-code/agent-side-coding/action-post\_response.md) message of an agent, we're really talking about a specific set of Dictionary key value pairs that have special meaning. All responses from the agent to the Mythic server already have to be in a structured format. Each of the above sections goes into what their reserved keywords mean, but a total list is found below:

* total\_chunks - integer - used with file uploads/downloads for chunking
* chunk\_num - integer - use with file uploads/downloads for chunking
* chunk\_size - integer - used with file uploads/downloads for chunking
* task\_id - string - UUID associated with tasks
* full\_path - string - used with file uploads/downloads to report back the full path of the file (ex: test.txt is meaningless, full\_path would report back C:\Users\username\Desktop\test.txt)
* user\_output - string - used with any command to display information back to the user
* completed - boolean - used with any command to indicate that the task is done (switches to the green completed icon)
* status - string - used to indicate that a command is not only done, but has encountered an error (value would be "error")
* file\_id - string (uuid to be specific) - used with file uploads/downloads and any command wishing to utilize chunking for files
* artifacts - array - an array of artifact objects that report back artifacts created on disk/on the network
* credentials - array - an array of credential objects that report back credentials gathered from the host
* window\_title - string - the title of the window associated with keystrokes for a keylogger
* user - string - the user associated with keystrokes for a keylogger
* keystrokes - string - the keystrokes for a keylogger
* edges - array - an array of P2P linking/unlinking information
* commands - array - an array of command updating information (telling mythic that the callback loaded/unloaded commands)
* file\_browser - dictionary - a dictionary of information about the file/folder with potentially a nested array to give details about all of the files within a folder
* process\_response - this is passed to your command's python file for processing in the `process_response` function.

## PayloadType Development Reference

As you're developing an agent to hook into these features, it's helpful to know where to look if you have questions. All of the Task, Command, and Parameter definitions/functions available to you are defined in the `mythic_payloadtype_container` PyPi Container, [MythicCommandBase.py](https://github.com/MythicMeta/Mythic\_PayloadType\_Container/blob/master/mythic\_payloadtype\_container/MythicCommandBase.py), which is hosted on the MythicMeta Organization on GitHub. Information about the Payload Type itself (BuildResponse, SupportedOS, BuildParameters, PayloadType, etc) can be found in the [PayloadBuilder.py](https://github.com/MythicMeta/Mythic\_PayloadType\_Container/blob/master/mythic\_payloadtype\_container/PayloadBuilder.py) file in the same PyPi repo.
