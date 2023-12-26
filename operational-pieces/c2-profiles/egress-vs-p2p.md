# Egress vs P2P

There are two kinds of C2 profiles - egress profiles that talk directly out of the target network or peer-to-peer (p2p) profiles that talk to neighboring agents.

## Egress Profiles

The [default HTTP](http.md) and the [dynamicHTTP](dynamichttp.md) profiles are both examples of egress profiles. They talk directly out of the target network. Egress profiles have associated Docker containers that allow you to do the translation between your special sauce c2 profile and the normal RESTful web requests to the main Mythic server. More information on how this works and how to create your own can be found here: [C2 Related Development](../../customizing/c2-related-development/).

## P2P Profiles

Peer-to-peer profiles in general are a bit different. They don't talk directly out to the internet; instead, they allow agents to talk to each other.&#x20;

This distinction between P2P and Egress for Mythic is made by a simple boolean indicating the purpose of the c2 container.

### P2P Visualizations

P2P profiles announce their connections to Mythic via [P2P Connections](../../customizing/hooking-features/linking-agents/action-p2p\_info.md). When Mythic gets these messages, it can start mapping out what the internal mesh looks like. To help view this from an operator perspective, there is an additional views on the main Callbacks page.

This view uses a directed graph to illustrate the connections between the agents. There's a central "Mythic Server" node that all egress agents connect to. When a route is announced, the view is updated to move one of the callbacks to be a child of another callback.

![](<../../.gitbook/assets/Screen Shot 2022-02-27 at 7.38.46 PM.png>)
