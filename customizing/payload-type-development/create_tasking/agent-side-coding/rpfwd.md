---
description: How does Reverse Port Forward work within Mythic
---

# RPFWD

## What is Reverse Port Forward?

Reverse port forwards provide a way to tunnel incoming connections on one port out to another IP:Port somewhere else. It normally provides a way to expose an internal service to a network that would otherwise not be able to directly access it.&#x20;

## What do RPFWD messages look like?

Agents transmit dictionary messages that look like the following:

```
{
  "exit": True,
  "server_id": 1234567,
  "data": "",
  "port": 80, // optional, but if you support multiple rpfwd ports within a single callback, you need this so Mythic knows which rpfwd you're getting data from
}
```

These messages contain three components:

* `exit` - boolean True or False. This indicates to either Mythic or your Agent that the connection has been terminated from one end and should be closed on the other end (after sending `data`). Because Mythic and 2 HTTP connections sit between the actual tool you're trying to proxy and the agent that makes those requests on your tool's behalf, we need this sort of flag to indicate that a TCP connection has closed on one side.
* `server_id` - unsigned int32. This number is how Mythic and the agent can track individual connections. Every new connection will generate a new `server_id` . Unlike SOCKS where Mythic is getting the initial connection, the agent is getting the initial connection in a reverse port forward. In this case, the agent needs to generate this random uint32 value to track connections.
* `data` - base64 string. This is the actual bytes that the proxied tool is trying to send.
* `port` - an optional uint32 value that specifies the port you're listening on within your agent. If your agent allows for multiple rpfwd commands within a single callback, then you need to specify this `port` so that Mythic knows _which_ rpfwd command this data is associated with and can redirect it out to the appropriate remote IP:Port combination. This `port` value is specifically the local port your agent is listening on, _not_ the port for the remote connection.

{% hint style="warning" %}
In Python translation containers, if `exit` is True, then `data` can be `None`
{% endhint %}

## How does this fit into Agent Messages?

These RPFWD messages are passed around as an array of dictionaries in `get_tasking` and `post_response` messages via a (added if needed) `rpfwd` key:

```
{
    "action": "get_tasking",
    "tasking_size": 1,
    "rpfwd": [
        {
            "exit": False,
            "server_id": 2,
            "data": "base64 string",
            "port": 80,
        },{
            "exit": True,
            "server_id": 1,
            "data": "",
            "port": 445,
        }
    ],
    "delegates": []
}
```

or in the `post_response` messages:

```
{
    "action": "post_response",
    "responses": [
        {
            "user_output": "blah",
            "task_id": "uuid here",
            "completed": true
        }
    ],
    "rpfwd": [
        {
            "exit": False,
            "server_id": 2,
            "data": "base64 string",
            "port": 80,
        },{
            "exit": True,
            "server_id": 1,
            "data": "",
            "port": 80,
        }
    ],
    "delegates": []
```

Notice that they're at the same level of "action" in these dictionaries - that's because they're not tied to any specific task, the same goes for delegate messages.

## How does an agent handle RPFWD?

For the most part, the message processing is pretty straight forward:

1. Agent opens port X on the target host where it's running
2. ServerA makes a connection to PortX
3. Agent accepts the connection, generates a new uint32 server\_id, and sends any data received to Mythic via `rpfwd` key. If the agent is tracking multiple ports, then it should also send the port the connection was received on with the message.
4. Mythic looks up the `server_id` (and optionally port) for that Callback if Mythic has seen this server\_id, then it can pass it off to the appropriate thread or channel to continue processing. If we've never seen the server\_id before, then it's likely a new connection that opened up, so we need to handle that appropriately. Mythic makes a new connection out to the RemoteIP:RemotePort specified when starting the `rpfwd` session. Mythic forwards the data along and waits for data back. Any data received is sent back via the `rpfwd` key the next time the agent checks in.
5. For existing connections, the agent looks at if `exit` is True or not. If `exit` is True, then the agent should close its corresponding TCP connection and clean up those resources. If it's not exit, then the agent should base64 decode the `data` field and forward those bytes through the existing TCP connection.
6. The agent should also be streaming data back from its open TCP connections to Mythic in its `get_tasking` and `post_response` messages.

That's it really. The hard part is making sure that you don't exhaust all of the system resources by creating too many threads, running into deadlocks, or any number of other potential issues.

While not perfect, the poseidon agent have a generally working implementation for Mythic: [https://github.com/MythicAgents/poseidon/blob/master/Payload\_Type/poseidon/poseidon/agent\_code/rpfwd/rpfwd.go](https://github.com/MythicAgents/poseidon/blob/master/Payload\_Type/poseidon/poseidon/agent\_code/rpfwd/rpfwd.go)
