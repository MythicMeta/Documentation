# Reverse PortFwd

See the [C2 Related Development section](../c2-related-development/c2-profile-code/agent-side-coding/rpfwd.md) for more RPFWD specific message details.

To start / stop RPFWD (or any interactive based protocol), use the `SendMythicRPCProxyStart` and `SendMythicRPCProxyStop` RPC calls within your Payload Type's tasking functions.

For RPFWD, you want to set `LocalPort` to the port you want to open up on the host where your agent is running. `RemoteIP` and `RemotePort` are used for Mythic to make remote connections based on the incoming connections your agent gets on `LocalPort` within the target network. The `PortType` will be `CALLBACK_PORT_TYPE_RPORTFWD` (i.e. `rpfwd`).
