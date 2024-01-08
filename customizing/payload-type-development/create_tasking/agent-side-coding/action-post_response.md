# 4. Submitting Responses

The main difference between submitting a response with a `post_response` and submitting responses with `get_tasking` is that in a `get_tasking` message with a `responses` key, you'll **also** get back additional tasking that's available. With a `post_response` message and a `responses` key, you _won't_ get back additional tasking that's ready for your agent. You can still get `socks`, `rpfwd`, `interact`, and `delegates` messages as part of your message back from Mythic, but you won't have a `tasks` key.&#x20;

## Message Request

The contents of the JSON message from the agent to Mythic when posting tasking responses is as follows:

```json
Base64( CallbackUUID + JSON(
{
	"action": "post_response",
	"responses": [
		{
			"task_id": "uuid of task",
			... response message (see below)
		},
		{
			"task_id": "uuid of task",
			... response message (see below)
		}
	], //if we were passing messages on behalf of other agents
	"delegates": [
		{"message": agentMessage, "c2_profile": "ProfileName", "uuid": "uuid here"},
		{"message": agentMessage, "c2_profile": "ProfileName", "uuid": "uuid here"}
		]
}
)
)
```

There are two things to note here:

* `responses` - This parameter is a list of all the responses for each tasking.
  * For each element in the responses array, we have a dictionary of information about the response. We also have a `task_id` field to indicate which task this response is for. After that though, comes the actual response output from the task.
    * If you don't want to hook a certain feature (like sending keystrokes, downloading files, creating artifacts, etc), but just want to return output to the user, the response section can be as simple as:\
      `{"task_id": "uuid of task", "user_output": "output of task here"}`
  * Each response style is described in [Hooking Features](../../../hooking-features/). The format described in each of the Hooking features sections replaces the `... response message` piece above
    * To continue adding to that JSON response, you can indicate that a command is finished by adding `"completed": true` or indicate that there was an error with `"status": "error"`.
* `delegates` - This parameter is not required, but allows for an agent to forward on messages from other callbacks. This is the peer-to-peer scenario where inner messages are passed externally by the egress point. Each of these messages is a self-contained "[Agent Message](agent-message-format.md)".

{% hint style="info" %}
Anything you put in `user_output` will go directly to the user to see. There's no additional processing that happens. If you want to perform additional processing on the response, then instead of `user_output` use the `process_response` key. This will allow you to perform additional processing on whatever is passed through the `process_response` key - from here, if you want to register something for the user to see, you'll need to use MythicRPCCreateResponse (you can use any MythicRPC at this point to register files, create credentials, etc).
{% endhint %}

## Message Response

Mythic responds with the following message format for post\_response requests:

```json
Base64( CallbackUUID + JSON(
{
	"action": "post_response",
	"responses": [
		{
			"task_id": UUID,
			"status": "success" or "error",
			"error": 'error message if it exists'
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

There are two things to note here:

* `responses` - This parameter is always a list and contains a success or error + error message for each task that was responded to.
* `delegates` - This parameter contains any responses for the messages that came through in the first message

{% hint style="info" %}
This message format also can take in `socks`, `rpfwd`, `interact`, `alerts`, `edges`, and `delegates` keys with their data as well. Just like with the `get_tasking` message, you can send all of that data along with each message.&#x20;
{% endhint %}
