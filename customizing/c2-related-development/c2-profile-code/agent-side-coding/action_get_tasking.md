---
description: This page describes the format for getting new tasking
---

# Action: get\_tasking

## Endpoint

All agent messages go to the same endpoint: `/api/v1.4/agent_message`

## Message Request

The contents of the JSON message from the agent to Mythic when requesting tasking is as follows:

```
Base64( CallbackUUID + JSON(
{
	"action": "get_tasking",
	"tasking_size": 1, //indicate the maximum number of tasks you want back
	//if passing on messages for other agents, include the following
	"delegates": [
		{"message": agentMessage, "c2_profile": "ProfileName", "uuid": "uuid here"},
		{"message": agentMessage, "c2_profile": "ProfileName", "uuid": "uuid here"}
	],
		"get_delegate_tasks": true, //optional, defaults to true
	}
)
)
```

There are two things to note here:

* `tasking_size` - This parameter defaults to one, but allows an agent to request how many tasks it wants to get back at once. If the agent specifies `-1` as this value, then Mythic will return _all_ of the tasking it has for that callback.
* `delegates` - This parameter is not required, but allows for an agent to forward on messages from other callbacks. This is the peer-to-peer scenario where inner messages are passed externally by the egress point. Each of these agentMessage is a self-contained "[Agent Message](agent-message-format.md)" and the `c2_profile` indicates the name of the C2 Profile used to connect the two agents. This allows Mythic to properly decode/translate the messages even for nested messages.
* `get_delegate_tasks` - This is an optional parameter. If you don't include it, it's assumed to be `True`. This indicates whether or not this `get_tasking` request should also check for tasks that belong to callbacks that are reachable from this callback. So, if agentA has a route to agentB, agentB has a task in the `submitted` state, and agentA issues a `get_tasking`, agentA can decide if it wants just its own tasking or if it also wants to pick up agentB's task as well.
  * Why does this matter? This is helpful if your linked agents issue their own periodic `get_tasking` messages rather than simply waiting for tasking to come to them. This way the parent callback (agentA in this case) doesn't accidentally consume and toss aside the task for agentB; instead, agentB's own periodic `get_tasking` message has to make its way up to Mythic for the task to be fetched.

## Message Response

Mythic responds with the following message format for get\_tasking requests:

```
Base64( CallbackUUID + JSON(
{
	"action": "get_tasking",
	"tasks": [
		{
			"command": "command name",
			"parameters": "command param string",
			"timestamp": 1578706611.324671, //timestamp provided to help with ordering
			"id": "task uuid",
		}
	],
	//if we were passing messages on behalf of other agents
	"delegates": [
		{"message": agentMessage, "c2_profile": "ProfileName", "uuid": "uuid here"},
		{"message": agentMessage, "c2_profile": "ProfileName", "uuid": "uuid here"}
	]
}
)
)
```

There are a few things to note here:

* `tasks` - This parameter is always a list, but contains between 0 and `tasking_size` number of entries.
* `parameters` - this encapsulates the parameters for the task. If a command has parameters like: `{"remote_path": "/users/desktop/test.png", "file_id": "uuid_here"}`, then the `params` field will have that JSON blob as a STRING value (i.e. the command is responsible to parse that out).&#x20;
* `delegates` - This parameter contains any responses for the messages that came through in the first message.&#x20;
