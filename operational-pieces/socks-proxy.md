# Socks Proxy

## What is it?

Socks proxy capabilities are a way to tunnel other traffic through another protocol. Within Mythic, this means tunneling other proxy-aware traffic through your normal C2 traffic. Mythic specifically leverages a modified Socks5 protocol without authentication (it's going through your C2 traffic afterall).

{% hint style="warning" %}
The Mythic server runs within a Docker container, and as such, you have to define which ports to expose externally. `Mythic/.env` has a special environment variable you can use to expose a range of ports at a time for this exact reason - `MYTHIC_SERVER_DYNAMIC_PORTS="7000-7010"`. By default this uses ports 7000-7010, but you can change this to any range you want and then simply restart Mythic to make the changes.
{% endhint %}

## Where is it?

Click the main Search field at the top and click the "Socks" icon on the far right (or click the socks icon at the top bar).

When you issue a command to start a socks proxy with Mythic, you specify an action "start/stop" and a port number. The port number you specify is the one you access remotely and leverage with your external tooling (such as proxychains).

## How does it work?

1. An operator issues a command to start socks on port 3333. This command goes to the associated payload type's container which does an RPC call to Mythic to open that port for Socks.
2. Mythic opens port 3333 in a go routine.
3. An operator configures proxychains to point to the Mythic server on port 3333.
4. An operator runs a tool through proxychains (ex: `proxychains curl https://www.google.com`)
5. Proxychains connects to Mythic on port 3333 and starts the Socks protocol negotiations.
6. The tool sends data through proxychains, and Mythic stores it in memory. In this temporary data, Mythic assigns each connection its own ID number.
7. The next time the agent checks in, Mythic takes this socks data and hands it off to the agent as part of the normal [Action: get\_tasking](../customizing/payload-type-development/create\_tasking/agent-side-coding/action\_get\_tasking.md) or [post\_response](../customizing/payload-type-development/create\_tasking/agent-side-coding/action-post\_response.md) process.
8. The agent checks if it's seen that ID before. If it has, it looks up the appropriate TCP connection and sends off the data. If it hasn't, it parses the Socks data to see where to open the connection. Then sends the resulting data and same randomID back to Mythic via [Action: post\_response](../customizing/payload-type-development/create\_tasking/agent-side-coding/action-post\_response.md).
9. Mythic gets the response, parses out the Socks specific data, and sends it back to proxychains

The above is a general scenario for how data is sent through for Socks. The Mythic server itself doesn't look at any of the data that's flowing - it simply tracks port to Callback mappings and shuttles data appropriately.

{% hint style="info" %}
Your proxy connections are at the mercy of the latency of your C2 channel. If your checkin time is every 10s, then you'll get one message of traffic sent every 20s (round trip time). This breaks a LOT of protocols. Therefore, it's recommended that you change the sleep of your agent down to something very low (0 or as close to it).
{% endhint %}

{% hint style="warning" %}
Don't forget to change the sleep interval of your agent _back_ to your normal intervals when you're done with Socks so that you reduce the burden on both the server and your agent.
{% endhint %}
