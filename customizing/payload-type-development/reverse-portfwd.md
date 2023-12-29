# 14. Reverse PortFwd

See the [C2 Related Development section](../c2-related-development/agent-side-coding/rpfwd.md) for more RPFWD specific message details.

To start / stop RPFWD (or any interactive based protocol), use the `SendMythicRPCProxyStart` and `SendMythicRPCProxyStop` RPC calls within your Payload Type's tasking functions.

For RPFWD, you want to set `LocalPort` to the port you want to open up on the host where your agent is running. `RemoteIP` and `RemotePort` are used for Mythic to make remote connections based on the incoming connections your agent gets on `LocalPort` within the target network. The `PortType` will be `CALLBACK_PORT_TYPE_RPORTFWD` (i.e. `rpfwd`).

In general, your agent will open up `LocalPort` on your target machine. When it gets a connection to that port, it'll generate a random uint32 to identify that connection, and forward along what it gets from the connection to Mythic along with that identifier. Mythic will get that information and open a new connection to `RemoteIP:RemotePort` and forward that data along. At that point, both sides are just forwarding data back and forth. Eventually, one end of the connection will terminate and at that point a final message will get sent to tell the other side to also close.
