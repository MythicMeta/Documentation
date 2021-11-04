# Actions

## What are actions?

Actions are special messages that don't adhere to the normal message types that you see for the rest of the features in this section. There are only a handful of these messages:

* [Action: Checkin](../c2-related-development/c2-profile-code/agent-side-coding/initial-checkin.md) - Initial checkin messages and key exchanges
* [Action: get\_tasking](../c2-related-development/c2-profile-code/agent-side-coding/action\_get\_tasking.md) - Getting tasking
* [Action: post\_response](../c2-related-development/c2-profile-code/agent-side-coding/action-post\_response.md) - Sending tasking responses
  * Inside of this is where the features listed throughout this section appear
* [Action: upload](../c2-related-development/c2-profile-code/agent-side-coding/action-upload.md) - Uploading files from Mythic to the agent
  * This is a special case. Downloading files from agent to Mythic happen within the post\_response messages, but moving files from Mythic to agent happens with the upload action
* Action: delegate - No other purpose than to shuttle delegate messages if the agent sending this message isn't doing any other action.
