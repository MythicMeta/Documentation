# Egress vs P2P

There are two kinds of C2 profiles - egress profiles that talk directly out of the target network or peer-to-peer (p2p) profiles that talk to neighboring agents.

## Egress Profiles

The [default HTTP](http.md) and the [dynamicHTTP](dynamichttp.md) profiles are both examples of egress profiles. They talk directly out of the target network. Egress profiles have associated Docker containers that allow you to do the translation between your special sauce c2 profile and the normal RESTful web requests to the main Mythic server. More information on how this works and how to create your own can be found here: [C2 Related Development](../customizing/c2-related-development/).

## P2P Profiles

Peer-to-peer profiles in general are a bit different. They don't talk directly out to the internet; instead, they allow agents to talk to each other. The standard scenario is to have at least one agent with _both_ an egress profile and a p2p profile which acts as a relay point for all of the other agents that only have p2p profiles.&#x20;

This distinction between P2P and Egress for Mythic is made by a simple boolean indicating the purpose of the c2 container. If you make a P2P profile, there's one additional piece of information that's needed - how messages are routed between these agents. "Server Routed" refers to the scenario where agents only know about their directly connected neighbors, so it's up to the "Server" to recursively craft a message that each agent can decrypt, read, pass along to a neighbor, which then in turn decrypts the message, reads it, and passes it along to one of its neighbors. This is sometimes referred to as "onion routing".&#x20;

When "Server Routed" is false, that means that the P2P mesh will handle its own routing. This is when the agents have their own way of routing messages between them that might periodically update, might be self recovering, or might be optimized over time. You can think of this like how normal networking is done with various network layer protocols.

### P2P Visualizations

P2P profiles announce their connections to Mythic via [P2P Connections](../customizing/hooking-features/action-p2p\_info.md). When Mythic gets these messages, it can start mapping out what the internal mesh looks like. To help view this from an operator perspective, there are two additional views on the main Callbacks page.

#### Graph View

This view uses a D3 force directed graph to illustrate the connections between the agents. There's a central "Mythic Server" node that all egress agents connect to. When a route is announced, the view is updated to move one of the callbacks to be a child of another callback.

![](<../.gitbook/assets/Screen Shot 2020-02-24 at 11.15.05 PM.png>)

#### Tree View

This view uses a D3 tree to illustrate the connections between the agents. There's a root "Mythic" node, and all other nodes are children off of that one. All egress C2 profiles are direct connections to the Mythic node, then agents connect up to the C2 profile they're using for egress communications. You can "right click" any node that has a light blue circle around it to hide or expand the tree from that point. If a node has a red circle around it, then it is high integrity, and if it has a yellow circle, then it's been removed from the table view. The reason these "removed" nodes can still be shown is when one node in a chain is removed, but there are still active nodes further down in the chain that still need to be shown.&#x20;

![](<../.gitbook/assets/Screen Shot 2020-07-13 at 12.44.24 PM.png>)
