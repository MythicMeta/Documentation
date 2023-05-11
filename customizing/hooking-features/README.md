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

When we talk about `Hooking Features` in the [Action: post\_response](../c2-related-development/c2-profile-code/agent-side-coding/action-post\_response.md) message of an agent, we're really talking about a specific set of Dictionary key value pairs that have special meaning. All responses from the agent to the Mythic server already have to be in a structured format. Each of the following sections goes into what their reserved keywords mean, but some simpler ones are:

* task\_id - string - UUID associated with tasks
* user\_output - string - used with any command to display information back to the user
* completed - boolean - used with any command to indicate that the task is done (switches to the green completed icon)
* status - string - used to indicate that a command is not only done, but has encountered an error or some other status to return to the user
* process\_response - this is passed to your command's python file for processing in the `process_response` function.

## PayloadType Development Reference

As you're developing an agent to hook into these features, it's helpful to know where to look if you have questions. All of the Task, Command, and Parameter definitions/functions available to you are defined in the `mythic_container` PyPi package, which is hosted on the MythicMeta Organization on GitHub. Information about the Payload Type itself (BuildResponse, SupportedOS, BuildParameters, PayloadType, etc) can be found in the [PayloadBuilder.py](https://github.com/MythicMeta/Mythic\_PayloadType\_Container/blob/master/mythic\_payloadtype\_container/PayloadBuilder.py) file in the same PyPi repo.
