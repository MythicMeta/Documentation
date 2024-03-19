---
description: How does SOCKS work within Mythic
---

# 5. SOCKS

## What is SOCKS?

Socks provides a way to negotiate and transmit TCP connections through a proxy ([https://en.wikipedia.org/wiki/SOCKS](https://en.wikipedia.org/wiki/SOCKS)). This allows operators to proxy network tools through the Mythic server and out through supported agents. SOCKS5 allows a lot more options for authentication compared to SOCKS4; however, Mythic currently doesn't leverage the authenticated components, so it's important that if you open up this port on your Mythic server that you lock it down.

{% hint style="warning" %}
Opened SOCKS5 ports in Mythic do not leverage additional authentication, so MAKE SURE YOU LOCK DOWN YOUR PORTS.
{% endhint %}

## What do SOCKS messages look like?

Without going into all the details of the SOCKS5 protocol, agents transmit dictionary messages that look like the following:

```
{
  "exit": True,
  "server_id": 1234567,
  "data": ""
}
```

These messages contain three components:

* `exit` - boolean True or False. This indicates to either Mythic or your Agent that the connection has been terminated from one end and should be closed on the other end (after sending `data`). Because Mythic and 2 HTTP connections sit between the actual tool you're trying to proxy and the agent that makes those requests on your tool's behalf, we need this sort of flag to indicate that a TCP connection has closed on one side.
* `server_id` - uint32. This number is how Mythic and the agent can track individual connections. Every new connection from a proxied tool (like through proxychains) will generate a new `server_id` that Mythic will send with data to the Agent.
* `data` - base64 string. This is the actual bytes that the proxied tool is trying to send.

{% hint style="warning" %}
In Python translation containers, if `exit` is True, then `data` can be `None`
{% endhint %}

## How does this fit into Agent Messages?

These SOCKS messages are passed around as an array of dictionaries in `get_tasking` and `post_response` messages via a (added if needed) `socks` key:

```
{
    "action": "get_tasking",
    "tasking_size": 1,
    "socks": [
        {
            "exit": False,
            "server_id": 2,
            "data": "base64 string"
        },{
            "exit": True,
            "server_id": 1,
            "data": ""
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
    "socks": [
        {
            "exit": False,
            "server_id": 2,
            "data": "base64 string"
        },{
            "exit": True,
            "server_id": 1,
            "data": ""
        }
    ],
    "delegates": []
```

Notice that they're at the same level of "action" in these dictionaries - that's because they're not tied to any specific task, the same goes for delegate messages.

{% hint style="info" %}
This means that if you send a `get_tasking` request OR a `post_response` request, you could get back `socks` data. The same goes for `rpfwd`, `interactive`, and `delegates`.&#x20;
{% endhint %}

## How does an agent handle SOCKS?

For the most part, the message processing is pretty straight forward:

1. Get a new SOCKS array
2. Get the first element from the list
3. If we know the `server_id`, then we can forward the message off to the appropriate thread or channel to continue processing. If we've never seen the server\_id before, then it's likely a new connection that opened up from an operator starting a new tool through proxychains, so we need to handle that appropriately.
4. For new connections, the first message is always a [SOCKS Request message](https://datatracker.ietf.org/doc/html/rfc1928#section-4) with encoded data for IP:PORT to connect to. This means that SOCKS authenticaion is already done. There's also a very specific message that gets sent back as a response to this. This small negotiation piece isn't something that Mythic created, it's just part of the [SOCKS protocol](https://datatracker.ietf.org/doc/html/rfc1928) to ensure that a tool like proxychains gets confirmation the agent was able to reach the desired IP:PORT
5. For existing connections, the agent looks at if `exit` is True or not. If `exit` is True, then the agent should close its corresponding TCP connection and clean up those resources. If it's not exit, then the agent should base64 decode the `data` field and forward those bytes through the existing TCP connection.
6. The agent should also be streaming data back from its open TCP connections to Mythic in its `get_tasking` and `post_response` messages.

That's it really. The hard part is making sure that you don't exhaust all of the system resources by creating too many threads, running into deadlocks, or any number of other potential issues.

While not perfect, the poseidon agent have a generally working implementation for Mythic: [https://github.com/MythicAgents/poseidon/blob/master/Payload\_Type/poseidon/poseidon/agent\_code/socks/socks.go](https://github.com/MythicAgents/poseidon/blob/master/Payload\_Type/poseidon/poseidon/agent\_code/socks/socks.go)
