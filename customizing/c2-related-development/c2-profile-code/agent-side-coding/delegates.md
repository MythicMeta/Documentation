# Delegates (p2p)

## What are delegate messages

Delegate messages are messages that an agent is forwarding on behalf of another agent. The use case here is an agent forwarding peer-to-peer messages for a linked agent. Mythic supports this by having an optional `delegates` array in messages. An example of what this looks like is in the next section, but this `delegates` array can be part of any message from an agent to mythic.

## Delegate parameter format

When sending delegate messages, there's a simple standard format:

#### Agent -> Mythic Message

```python
{
    "action": "some action here",
    "delegates": [
        {
            "message": agentMessage,
            "uuid": UUID,
            "c2_profile": "ProfileName"
        }
    ]
}
```

Within a delegates array are a series of JSON dictionaries:

* `UUID` - This field is some UUID identifier used by the agent to track where a message came from and where it should go back to. Ideally this is the same as the UUID for the callback on the other end of the connection, but can be any value. If the agent uses a value that does not match up with the UUID of the agent on the other end, Mythic will indicate that in the response. This allows the middle-man agent to generate some UUID identifier as needed upon first connection and then learn of and use the agent's real UUID once the messages start flowing.
* `message` - this is the actual message that the agent is transmitting on behalf of the other agent
* `c2_profile` - This field indicates the name of the C2 Profile associated with the connection between this agent and the delegated agent. This allows Mythic to know how these two agents are talking to each other when generating and tracking connections.

#### Mythic -> Agent Reply

```python
{
    "action": "some action here",
    "delegates": [
        {
            "message": agentMessage,
            "uuid": "same UUID as the message agent -> mythic",
            "mythic_uuid": UUID that mythic uses
        }
    ]
}
```

The `mythic_uuid` field indicates that the `uuid` field the agent sent doesn't match up with the UUID in the associated message. If the agent uses the right UUID with the agentMessage then the response would be:

```python
{
    "action": "some action here",
    "delegates": [
        {
            "message": agentMessage,
            "uuid": "same UUID as the message agent -> mythic"
        }
    ]
}
```

Why do you care and why is this important? This allows an agent to randomly generate its own UUID for tracking connections with other agents and provides a mechanism for Mythic to reveal the right UUID for the callback on the other end. This implicitly gives the agent the right UUID to use if it needs to announce that it lost the route to the callback on the other end. If Mythic didn't correct the agent's use of UUID, then when the agent loses connection to the P2P agent, it wouldn't be able to properly indicate it to Mythic.

## Example walkthrough

Ok, so let's walk through an example:

* agentA is an egress agent speaking HTTP to Mythic. agentA sends messages directly to Mythic, such as the `{"action": "get_tasking", "tasking_size": 1}`. All is well.
* somehow agentB gets deployed and executed, this agent (for sake of example) opens a port on its host (same host as agentA or another one, doesn't matter)
* agentA connects to agentB (or agentB connects to agentA if agentA opened the port and agentB did a connection to it) over this new P2P protocol (smb, tcp, etc)
* agentB sends to agentA a staging message if it's doing EKE, a checkin message if it's already an established callback (like the example of re-linking to a callback), or a checkin message if it's doing like a static PSK or plaintext. The format of this message is exactly the same as if it wasn't going through agentA
* agentA gets this message, and is like "new connection, who dis?", so it makes a random UUID to identify whomever is on the other end of the line and forwards that message off to Mythic with the next message agentA would be sending anyway. So, if the next message that agentA would send to Mythic is another get tasking, then it would look like: `{"action": "get_tasking", "tasking_size": 1, "delegates": [ {"message": agentB's message, "c2_profile": "Name of the profile we're using to communicate", "uuid": "myRandomUUID"} ] }`. That's the message agentA sends to Mythic.
* Mythic gets the message, processes the get\_tasking for agentA, then sees it has `delegate` messages (i.e. messages that it's passing along on behalf of other agents). So Mythic recursively processes each of the messages in this array. Because that `message` value is the same as if agentB was talking directly to Mythic, Mythic can parse out the right UUIDs and information. The `c2_profile` piece allows Mythic to look up any c2-specific encryption information to pass along for the message. Once Mythic is done processing the message, it sends a response back to agentA like: `{"action": "get_tasking", "tasks": [ normal array of tasks ], "delegates": [ {"message": "response back to what agentB sent", "uuid": "myRandomUUID that agentA generated", "mythic_uuid": "the actual UUID that Mythic uses for agentB"} ] }`. If this is the first time that Mythic has seen a delegate from agentB through agentA, then Mythic knows that there's a route between the two and via which C2 profile, so it can automatically display that in the UI
* agentA gets the response back, processes its get\_tasking like normal, sees the `delegates` array and loops through those messages. It sees "oh, it's myRandomUUID, i know that guy, let me forward it along" and also sees that it's been calling agentB by the wrong name, it now knows agentB's real name according to Mythic. This is important because if agentA and agentB ever lose connection, agentA can report back to Mythic that it can no longer to speak to agentB with the right UUID that Mythic knows.

This same process repeats and just keeps nesting for agentC that would send a message to agentB that would send the message to agentA that sends it to Mythic. agentA can't actually decrypt the messages between agentB and Mythic, but it doesn't need to. It just has to track that connection and shuttle messages around.

Now that there's a "route" between the two agents that Mythic is aware of, a few things happen:

* when agentA now does a `get_tasking` message (with or without a delegate message from agentB), if mythic sees a tasking for agentB, Mythic will automatically add in the same `delegates` message that we saw before and send it back with agentA so that agentA can forward it to agentB. That's important - agentB never had to ask for tasking, Mythic automatically gave it to agentA because it knew there was a route between the two agents.
* if you DON"T want that to happen though - if you want agentB to keep issuing get\_tasking requests through agentA with periodic beaconing, then in agentA's get\_tasking you can add `get_delegate_tasks` to False. i.e (`{"action": "get_tasking", "tasking_size": 1, "get_delegate_tasks": false}`) then even if there are tasks for agentB, Mythic WILL NOT send them along with agentA. agentB will have to ask for them directly

What happens when agentA and agentB can no longer communicate though? agentA needs to send a message back to Mythic to indicate that the connection is lost. This can be done with the [routes](../../../hooking-features/action-p2p\_info.md) key. Using all of the information agentA has about the connection, it can announce that Mythic should remove a edge between the two callbacks. This can either happen as part of a response to a tasking (such as an explicit task to unlink two agents) or just something that gets noticed (like a computer rebooted and now the connection is lost). In the first case, we see the example below as part of a normal post\_response message:

```python
{
 "user_output": "some ouser output here",
 "task_id": "uuid of task here",
 "edges": [
    {
      "source": "uuid of source callback",
      "destination": "uuid of destination callback",
      "direction": 1,
      "metadata": "",
      "action": "remove"
      "c2_profile": "name of the c2 profile used in this connection"
     }
   ]
}
```

If this wasn't part of some task, then there would be no task\_id to use. In this case, we can add the same `edges` structure at a higher point in the message:

```python
{
 "action": "get_tasking" (could be "post_response", "upload", etc)
 "edges": [
    {
      "source": "uuid of source callback",
      "destination": "uuid of destination callback",
      "direction": 1 or 2 or 3,
      "metadata": "{ optional metadata json string }",
      "action": "add" or "remove"
      "c2_profile": "name of the c2 profile used in this connection"
     }
   ]
}
```
