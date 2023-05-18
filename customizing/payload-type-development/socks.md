# SOCKS

See the [C2 Related Development section](../c2-related-development/c2-profile-code/agent-side-coding/socks.md) for more SOCKS specific message details.

To start / stop SOCKS (or any interactive based protocol), use the `SendMythicRPCProxyStart` and `SendMythicRPCProxyStop` RPC calls within your Payload Type's tasking functions.

For SOCKS, you want to set `LocalPort` to the port you want to open up on the Mythic Server - this is where you'll point your proxy-aware tooling (like proxychains) to then tunnel those requests through your C2 channel and out your agent. For SOCKS, the RemotePort and RemoteIP don't matter. The `PortType` will be `CALLBACK_PORT_TYPE_SOCKS` (i.e. `socks`).
