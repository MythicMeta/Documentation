---
description: This page describes how an agent message is formatted
---

# Agent Message Format

All messages go to the `/api/v1.4/agent_message` endpoint via the associated C2 Profile docker container. These messages can be:

* POST request
  * message content in body
  * message content in FIRST header value
  * message content in FIRST cookie value
* GET request
  * message content in FIRST header value
  * message content in FIRST cookie value
  * message content in FIRST query parameter

All agent messages have the same general structure, but it's the message inside the structure that varies.

Each message has the following general format:

```python
base64(
	UUID + EncBlob( //the following is all encrypted
		JSON({
			"action": "", //indicating what the message is - required
			"...": ... // JSON data relating to the action - required
			
			//this piece is optional and just for p2p mesh forwarding
			"delegates": [
			{"message": agentMessage, "c2_profile": "ProfileName", "uuid": "uuid here"},
			{"message": agentMessage, "c2_profile": "ProfileName", "uuid": "uuid here"}
			]
		})
	)
)
```

There are a couple of components to note here in what's called an `agentMessage`:

* `UUID` - This UUID varies based on the phase of the agent (initial checkin, staging, fully staged). This is a 36 character long of the format `b50a5fe8-099d-4611-a2ac-96d93e6ec77b` . Optionally, if your agent is dealing with more of a binary-level specification rather than strings, you can use a 16 byte little-endian value here for the binary representation of the UUID4 string.
* `EncBlob` - This section is encrypted, typically by an AES256 key, but when agents are staging, this could be encrypted with RSA keys or as part of some other custom crypto/staging you're doing as part of your payload type container. .
* `JSON` - This is the actual message that's being sent by the agent to Mythic or from Mythic to an agent. If you're doing your own custom message format and leveraging a translation container, this this format will obviously be different and will match up with your custom version; however, in your translation container you will need to convert back to this format so that Mythic can process the message.
  * `action` - This specifies what the rest of the message means. This can be one of the following:
    * staging\_rsa
    * checkin
    * get\_tasking
    * post\_response
    * upload
    * delegate
    * translation\_staging (you're doing your own staging)
  * `...` - This section varies based on the action that's being performed. The different variations here can be found in [Hooking Features](../../../hooking-features/) , [Initial Checkin](initial-checkin.md), and [Agent Responses](action\_get\_tasking.md)
  * `delegates` - This section contains messages from other agents that are being passed along. This is how messages from nested peer-to-peer agents can be forwarded out through and egress callback. If your agent isn't forwarding messages on from others (such as in a p2p mesh or as an egress point), then you don't need this section. More info can be found here: [Delegates (p2p)](delegates.md)
* `+` - when you see something like `UUID + EncBlob`, that's referring to byte concatenation of the two values. You don't need to do any specific processing or whatnot, just right after the first elements bytes put the second elements bytes

## Message Format for Custom Agent Messages

If you want to have a completely custom agent message format (different format for JSON, different field names/formatting, a binary or otherwise formatted protocol, etc), then there's only two things you have to do for it to work with Mythic.

1. Base64 encode the message
2. The first bytes of the message must be the associated UUID (payload, staging, callback).

Mythic uses these first few bytes to do a lookup in its database to find out everything about the message. Specifically for this case, it looks up if the associated payload type has a translation container, and if so, ships the message off to it first before trying to process it.
