# A note about containers

## Mythic and containers

Mythic uses docker containers to logically separate different components and functions. There are three main categories:

1. Mythic's main core. This consists of three docker containers stood up with docker-compose:
   1. mythic\_server - An async python3 Sanic instance
   2. mythic\_postgres - An instance of a postgresql database
   3. mythic\_rabbitmq - An instance of a rabbitmq container for message passing between containers
   4. mythic\_nginx - A instance of a reverse Nginx proxy
   5. mythic\_graphql - An instance of a Hasura GraphQL server
   6. mythic\_redis - An instance of a small redis database for use with JWT storage and SOCKS data
2. Command and Control (C2) Profiles
   1. Any folder in `Mythic/C2_Profiles` will be treated like a docker container specific to C2 profiles (more on this in the C2 section).
3. Payload Types
   1. Any folder in `Mythic/Payload_Types` will be treated like a docker container specific to a payload type (such as apfell, atlas, etc)

To stop a specific C2 Profile's container, run `sudo ./mythic-cli c2 stop {c2_profile_name}` or `sudo ./mythic-cli c2 stop` to stop all of them.

To stop a specific Payload Type's container, run `sudo ./mythic-cli payload stop {payload_type_name}` or `sudo ./mythic-cli payload stop` to stop all of them.

If you want to reset all of the data in the database, use `sudo ./mythic-cli database reset`.&#x20;

If you want to start/restart any specific payload type container, you can do `sudo ./mythic-cli payload start {payload_type_name}` and just that container will start/restart. If you want to start multiple, just do spaces between them: `sudo ./mythic-cli payload start {payload type 1} {payload type 2}`.&#x20;

{% hint style="info" %}
Mythic shares the networking with the host it's on. This allows Mythic to not worry about exposing specific ports ahead of time for each container since they can be dynamically set by users. However, this does mean that Mythic needs to run as `root` if any ports under 1024 need to be used.
{% endhint %}

### Docker-compose

All of Mythic's containers now share a single docker-compose file. When you install an agent or C2 Profile this docker-compose file will automatically be updated. However, you can always add/remove from this file via `mythic-cli` and list out what's registered in the docker-compose file vs what you have available on your system:

```
./mythic-cli payload add apfell
[+] Successfully updated docker-compose.yml

./mythic-cli payload list
Docker-compose entries:
[+] apfell

Payload Types on disk:
[+] apfell
[+] atlas
[+] leviathan
[+] poseidon
[+] service_wrapper

/mythic-cli c2 add http
[+] Successfully updated docker-compose.yml

./mythic-cli c2 list
Docker-compose entries:
[+] http

C2 Profiles on disk:
[+] dynamichttp
[+] http
[+] websocket
```

This makes it easy to track what's available to you and what you're currently using.

## Architecture

Mythic's architecture is broken out in the following diagram:

![](<../.gitbook/assets/Screen Shot 2020-08-10 at 12.58.03 PM.png>)

Operators connect via a browser to the main Mythic server. This server is a `Sanic`, python3 web server. This main Mythic server connects to a PostgreSQL database where information about the operations lives. Each of these are in their own docker containers. When Mythic needs to talk to any payload type container or c2 profile container, it does so via RabbitMQ, which is in its own docker container as well.&#x20;

When an agent calls back, it connects through these c2 profile containers which have the job of transforming whatever the c2 profile specific language/style is back into the normal RESTful API calls that the Mythic server needs.&#x20;

## Mythic Container Locations

The base containers for the Mythic agents are located with the `itsafeaturemythic` DockerHub repository: [https://hub.docker.com/search?q=itsafeaturemythic\&type=image](https://hub.docker.com/search?q=itsafeaturemythic\&type=image).&#x20;

There are currently base containers for csharp, python3.8, and a special one for poseidon's xgo environment. More containers will be added in the future, but having the base containers pre-configured on DockerHub speeds up install time for operators.
