# A note about containers

## Mythic and containers

Mythic uses docker containers to logically separate different components and functions. There are three main categories:

1. Mythic's main core. This consists of docker containers stood up with docker-compose:
   1. mythic\_server - An GoLang gin webserver instance
   2. mythic\_postgres - An instance of a postgresql database
   3. mythic\_rabbitmq - An instance of a rabbitmq container for message passing between containers
   4. mythic\_nginx - A instance of a reverse Nginx proxy
   5. mythic\_graphql - An instance of a Hasura GraphQL server
   6. mythic\_jupyter - An instance of a Jupyter notebook
   7. mythic\_documentation - An instance of a Hugo webserver for localized documentation
2. Installed Services
   1. Any folder in `Mythic/InstalledServices` will be treated like a docker container (payload types, c2 profiles, webhooks, loggers, translation containers, etc)&#x20;

To stop a specific container, run `sudo ./mythic-cli stop {c2_profile_name}` .

If you want to reset all of the data in the database, use `sudo ./mythic-cli database reset`.

If you want to start/restart any specific payload type container, you can do `sudo ./mythic-cli start {payload_type_name}` and just that container will start/restart. If you want to start multiple, just do spaces between them: `sudo ./mythic-cli start {container 1} {container 2}`.

{% hint style="info" %}
Mythic shares the networking with the host it's on. This allows Mythic to not worry about exposing specific ports ahead of time for each container since they can be dynamically set by users. However, this does mean that Mythic needs to run as `root` if any ports under 1024 need to be used.
{% endhint %}

### Docker-compose

All of Mythic's containers share a single docker-compose file. When you install an agent or C2 Profile this docker-compose file will automatically be updated. However, you can always add/remove from this file via `mythic-cli` and list out what's registered in the docker-compose file vs what you have available on your system:

```
./mythic-cli add apfell
[+] Successfully updated docker-compose.yml

/mythic-cli remove http
[+] Successfully updated docker-compose.yml
```

This makes it easy to track what's available to you and what you're currently using.

## Architecture

Mythic's architecture is broken out in the following diagram:

Operators connect via a browser to the main Mythic server, a GoLang `gin` web server. This main Mythic server connects to a PostgreSQL database where information about the operations lives. Each of these are in their own docker containers. When Mythic needs to talk to any payload type container or c2 profile container, it does so via RabbitMQ, which is in its own docker container as well.

When an agent calls back, it connects through these c2 profile containers which have the job of transforming whatever the c2 profile specific language/style is back into the normal RESTful API calls that the Mythic server needs.

## Mythic Container Locations

The base containers for the Mythic agents are located with the `itsafeaturemythic` DockerHub repository: [https://hub.docker.com/search?q=itsafeaturemythic\&type=image](https://hub.docker.com/search?q=itsafeaturemythic\&type=image).

There is currently a base container that contains Python 3.11, Mono, .Net Core 7.0, Golang 1.20, and the macOS 12.1 SDK. More containers will be added in the future, but having the base containers pre-configured on DockerHub speeds up install time for operators.
