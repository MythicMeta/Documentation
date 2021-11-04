---
description: This describes how to report back p2p connection information to the server
---

# P2P Connections

## What is it

This message type allows agents to report back new or removed connections between themselves or elsewhere within a p2p mesh. Mythic uses these messages to construct a graph of connectivity that's displayed to the user and for handling routing for messages through the mesh.

### Agent message to Mythic

The agent message to Mythic has the following form:

```python
{
 "user_output": "some ouser output here",
 "task_id": "uuid of task here",
 "edges": [
    {
      "source": "uuid of source callback",
      "destination": "uuid of destination callback",
      "metadata": "{ optional metadata json string }",
      "action": "add" or "remove"
      "c2_profile": "name of the c2 profile used in this connection"
     }
   ]
}
```

Just like other [post\_response](../c2-related-development/c2-profile-code/agent-side-coding/action-post\_response.md) messages, this message has the same UUID and encryption requirements found in [Agent Message Format](../c2-related-development/c2-profile-code/agent-side-coding/agent-message-format.md). Some things to note about the fields:

* `edges` is an array of JSON objects describing the state of the connections that the agent is adding/removing. Each edge in this array has the following fields:
  * `source` this is one end of the p2p connection (more often than not, this is the agent that's reporting this information)
  * `destination` this is the other end of the p2p connection
  * `metadata` is additional information about the connection that the agent wants to report. For example, when dealing with SMB bind pipes, this could contain information about the specific pipe name instances that are being used if they're being programmatically generated.
  * `action` this indicates if the connection described above is to be added or removed from Mythic.
  * `c2_profile` this indicates which c2 profile is used for the connection

### Response message from Mythic

After getting a message like this, Mythic responds with a message of the following form:

```python
{
    "status": "success" or "error",
    "error": "error message if status was error",
    "task_id": "id of task"
}
```

This is very similar to most other response messages from the Mythic server.

### Automatic Connection Announcements

When an agent sends a message to Mythic with a `delegate` component, Mythic will automatically add a route between the delegate and the agent that sent the message.&#x20;

For example: If agentA is an egress agent sending messages to a C2 Docker container and it links to agentB, a p2p agent. There are now a few options:

* If the p2p protocol involves sending messages back-and-forth between the two agents, then agentA can determine if agentB is a new payload, trying to stage, or a complete callback. When agentB is a complete callback, agentA can announce a new route to agentB.
* When agentA sends a message to Mythic with a delegate message from agentB, Mythic will automatically create a route between the two agents.&#x20;

This distinction is important due to how the p2p protocol might work. It could be the case that agentB never sends a `get_tasking` request and simply waits for messages to it. In this case, agentA would have to do some sort of p2p comms with agentB to determine who it is so it can announce the route or start the staging process for agentB.
