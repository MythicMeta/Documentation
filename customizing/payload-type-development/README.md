---
description: This section describes new Payload Types
---

# Payload Type Development

## Creating a new Mythic agent

You want to create a new agent that fully integrates with Mythic. Since everything in Mythic revolves around Docker containers, you will need to ultimately create one for your payload type along with some specific files/folder structures. This can be done with docker containers on the same host as the Mythic server or with an [external VM/host machine](./#turning-a-vm-into-an-apfell-container).

* Optionally: You can also have a custom C2 profile that this agent supports, see [C2 Profile Development](../c2-related-development/) for that piece though

{% hint style="info" %}
The first step is to look into the [First Steps](first-steps/) page to get everything generally squared away and look at the order of things that need your attention. The goal is to get you up and going as fast as possible!
{% endhint %}

## Payload/C2 Development Resources

There are a lot of things involved in making your own agent, c2 profile, or translation container. For the bare-bones implementations, you'll have to do very little, but Mythic tries to allow a lot of customizability. To help with this, Mythic uses PyPi packages for a lot of its configuration/connections and Docker images for distribution. This makes it easy for somebody to just kind of "hook up" and get going. If you don't want to do that though, all of this code is available to you within the MythicMeta organization on GitHub ([https://github.com/MythicMeta](https://github.com/MythicMeta)).

This organization is broken out in five main repos:

* `Mythic_Scripting` - This holds all of the code for the PyPi package, `mythic`, that you can use to script up actions.
* `MythicContainerPyPi` - This holds all of the code for the PyPi package
* `MythicContainer` - This holds all of the same sort of code as the PyPi package, but is for agents that wish to define commands in Golang instead of Python.

## Payload Type Docker Information

The rest of these sections talk about specifics for the docker container for your new agent.

### Creating a docker container for your new agent

There are base docker images for the various kinds of environments you're likely to need:

* `itsafeaturemythic/mythic_go_base` has GoLang 1.21 installed
* `itsafeaturemythic/mythic_go_dotnet` has GoLang 1.21 and .NET
* `itsafeaturemythic/mythic_go_macos` has GoLang 1.21 and the macOS SDK
* `itsafeaturemythic/mythic_python_base` has Python 3.11 and the `mythic_container` pypi package
* `itsafeaturemythic/mythic_python_go` has Python 3.11, the `mythic_container` pypi package, and GoLang v1.21
* `itsafeaturemythic/mythic_python_macos` has Python 3.11, the `mythic_container` pypi package, and the macOS SDK

This allows two payload types that might share the same language to still have different environment variables, build paths, tools installed, etc. Docker containers come into play for a few things:

* Metadata about the payload type (this is in the form of python classes or Golang structs)
* The payload type code base (whatever language your agent is in)
* The code to create the payload based on all of the user supplied input
* Metadata about all of the commands associated with that payload type
* The code for all of those commands (whatever language your agent is in)
* Browser scripts for commands and support scripts for the payload type as a whole (JavaScript)
* The code to take user supplied tasking and turn it into tasking for your agent

{% hint style="warning" %}
This part has to happen outside of the web UI at the moment. The web UI is running in its own docker container, and as such, can't create, start, or stop entire docker containers. So, make sure this part is done via CLI access to where Mythic is installed.
{% endhint %}

Within `Mythic/InstalledServices/` make a new folder that matches the name of your agent. Inside of this folder make a file called `Dockerfile`. This is where you have a choice to make - either use the default docker container as a starting point and make your additions from there, or use a different container base.

#### Using the default container base

&#x20;Start your `Dockerfile` off with one of the above images:

```
From itsafeaturemythic/mythic_python_base:latest
```

On the next lines, just add in any extra things you need for your agent to properly build, such as:

```
RUN pip install python_module_name
RUN shell_command
RUN apt-get install -y tool_name
```

{% hint style="info" %}
This happens all in a script for docker, so if a command might make a prompt for something (like apt-get), make sure to auto handle that or your stuff won't get installed properly
{% endhint %}

The latest container versions and their associated `mythic_container` PyPi versions can be found here: [#current-payloadtype-versions](first-steps/container-syncing.md#current-payloadtype-versions "mention").

If you're curious what else goes into these containers, look in the `docker-templates` folder within the Mythic repository.

#### Required Folder Structure

{% hint style="info" %}
The `Mythic/InstalledServices/[agent name]` folder is mapped to `/Mythic` in the docker container. Editing the files on disk results in the edits appearing in the docker container and visa versa.
{% endhint %}

Most things are left up to you, the agent developer, to decide how best to structure your code and agent code. However, there's a few files that need to be in certain places for Mythic's Docker container to properly kick off your container:

1. `Mythic/InstalledServices/[agent name]/Dockerfile` <-- this defines the docker information for your agent and must be here for docker-compose to properly discover and build your container

Technically, that's all that's required. Within the `Dockerfile` you will then need to do whatever is needed to kick of your main program that imports either the MythicContainer PyPi package or the MythicContainer Golang package. As some examples, here's what you can do for Python and Golang:

#### Python:

1. `Mythic/InstalledServices/[agent name]/main.py` <-- if you plan on using Python as your definition language, this `main.py` file is what will get executed by Python 3.11 assuming you use the Dockerfile shown below. If you want a different structure, just change the `CMD` line to execute whatever it is you want.

At that point, your `main.py` file will import any other folders/files needed to define your agent/commands and import the `mythic_container` pypi package. Here's an example from Apollo:

```
FROM itsafeaturemythic/mythic_python_base:latest

RUN python3 -m pip install donut-shellcode

WORKDIR /Mythic/

CMD ["python3", "main.py"]
```

#### Golang

Things are a little different here as we're going from source code to compiled binaries. To keep things in a simplified area for building, running, and testing, a common file like a `Makefile` is useful.&#x20;

1. `Mythic/InstalledServices/[agent name]/Makefile`&#x20;

From here, that make file can have different functions for what you need to do:

Here's an example of the `Makefile` that allows you to specify custom environment variables when debugging locally, but also support Docker building:

```
BINARY_NAME?=main
DEBUG_LEVEL?="warning"
RABBITMQ_HOST?="127.0.0.1"
RABBITMQ_PASSWORD?="password here"
MYTHIC_SERVER_HOST?="127.0.0.1"
MYTHIC_SERVER_GRPC_PORT?="17444"
WEBHOOK_DEFAULT_URL?=
WEBHOOK_DEFAULT_CHANNEL?=
WEBHOOK_DEFAULT_FEEDBACK_CHANNEL?=
WEBHOOK_DEFAULT_CALLBACK_CHANNEL?=
WEBHOOK_DEFAULT_STARTUP_CHANNEL?=

build:
	go mod tidy
	GOOS=linux go build -o ${BINARY_NAME} .
	cp ${BINARY_NAME} /

run:
	cp /${BINARY_NAME} .
	./${BINARY_NAME}

run_custom:
	DEBUG_LEVEL=${DEBUG_LEVEL} \
RABBITMQ_HOST=${RABBITMQ_HOST} \
RABBITMQ_PASSWORD=${RABBITMQ_PASSWORD} \
MYTHIC_SERVER_HOST=${MYTHIC_SERVER_HOST} \
MYTHIC_SERVER_GRPC_PORT=${MYTHIC_SERVER_GRPC_PORT} \
WEBHOOK_DEFAULT_URL=${WEBHOOK_DEFAULT_URL} \
WEBHOOK_DEFAULT_CHANNEL=${WEBHOOK_DEFAULT_CHANNEL} \
WEBHOOK_DEFAULT_FEEDBACK_CHANNEL=${WEBHOOK_DEFAULT_FEEDBACK_CHANNEL} \
WEBHOOK_DEFAULT_CALLBACK_CHANNEL=${WEBHOOK_DEFAULT_CALLBACK_CHANNEL} \
WEBHOOK_DEFAULT_STARTUP_CHANNEL=${WEBHOOK_DEFAULT_STARTUP_CHANNEL} \
./${BINARY_NAME}

```

{% hint style="info" %}
Pay attention to the `build` and `run` commands - once you're done building your code, notice that it's copied from the current directory to `/` in the Docker Image. This is because when the container starts, your source code is mapped into the Docker image, thus discarding any changes you made to that directory while building. This is also why the `run` function copies the binary back into the current directory and executes it there. The reason it's executed this way instead of from `/` is so that pathing and local folders are located where you expect them to be in relation to your binary.
{% endhint %}

To go along with that, a sample Docker file for Golang is as follows:

```
FROM itsafeaturemythic/mythic_go_base:latest

WORKDIR /Mythic/

COPY [".", "."]

RUN make build

CMD make run
```

It's very similar to the Python version, except it runs `make build` when building and `make run` when running the code. The Python version doesn't need a `Makefile` or multiple commands because it's an interpreted language.

#### Custom container

There isn't _too_ much that's different if you're going to use your own container, it's just on you to make sure that python3.11/Golang 1.21 is up and running and the entrypoint is set properly.&#x20;

{% hint style="info" %}
If you're having this hook up through Mythic via `mythic-cli` and the one `docker-compose` file, the `dynaconf` and `mythic-container` (or MythicContainer GoLang package) are the ones responsible for the `Mythic/.env` configuration being applied to your container.
{% endhint %}

{% hint style="warning" %}
If your container/service is running on a different host than the main Mythic instance, then you need to make sure the `rabbitmq_password` is shared over to your agent as well. By default, this is a randomized value stored in the `Mythic/.env` file and shared across containers, but you will need to manually share this over with your agent either via an environment variable (`MYTHIC_RABBITMQ_PASSWORD` ) or by editing the `rabbitmq_password` field in your rabbitmq\_config.json file.
{% endhint %}

### Starting your Docker container

To start your new payload type docker container, you need to first make sure that the docker-compose file is aware of it (assuming you didn't install it via `mythic-cli install github <url>` ). You can simply do `mythic-cli add <payload name>`. Now you can start just that one container via `mythic-cli start <payload name>`.

Your container should pull down all the necessary files, run the necessary scripts, and finally start. If it started successfully, you should see it listed in the payload types section when you run `sudo ./mythic-cli status`.

If you go back in the web UI at this point, you should see the red text next to your payload type change to green to indicate that it's now running. If it's not, then something went wrong along the way. You can use the `sudo ./mythic-cli logs payload_type_name` to see the output from the container to potentially troubleshoot.

{% hint style="info" %}
The containers will automatically sync all of their information with the Mythic server when they start, so the first time the Mythic server gets a message from a container it doesn't know about, it'll ask to sync. Similarly, as you do development and restart your Payload Type container, updates will automatically get synced to the main UI.
{% endhint %}

If you want to go interactive within the container to see what's up, use the following command:

```
sudo docker exec -it {payload type name} /bin/bash
```

The container has to be running though.

## Turning a VM into a Mythic container

There are scenarios in which you need a Mythic container for an agent, but you can't (or don't want) to use the normal docker containers that Mythic uses. This could be for reasons like:

* You have a custom build environment that you don't want to recreate
* You have specific kernel versions or operating systems you're wanting to develop with

So, to leverage your own custom VM or physical computer into a Mythic recognized container, there are just a few steps.

{% hint style="warning" %}
External agents need to connect to `mythic_rabbitmq` in order to send/receive messages. They also need to connect to the `mythic_server` to transfer files and potentially use gRPC. By default, these container is bound on localhost only. In order to have an external agent connect up, you will need to adjust this in the `Mythic/.env` file to have `RABBITMQ_BIND_LOCALHOST_ONLY=false` and `MYTHIC_SERVER_BIND_LOCALHOST_ONLY=false` and restart Mythic (`sudo ./mythic-cli restart`).&#x20;
{% endhint %}

1. Install python 3.10+ (or Golang 1.21) in the VM  or on the computer
2. `pip3 install mythic-container` (this has all of the definitions and functions for the container to sync with Mythic and issue RPC commands). Make sure you get the right version of this PyPi package for the version of Mythic you're using ([#current-payloadtype-versions](first-steps/container-syncing.md#current-payloadtype-versions "mention")). Alternatively, `go get -u github.com/MythicMeta/MythicContainer` for golang.
3. Create a folder on the computer or VM (let's call it path `/pathA`). Essentially, your `/pathA` path will be the new `InstalledServices/[agent name]` folder. Create a sub folder for your actual agent's code to live, like `/pathA/agent_code`. You can create a Visual Studio project here and simply configure it however you need.
4. Your command function definitions and payload definition are also helpful to have in a folder, like `/pathA/agent_functions`.
5.  Edit the `/pathA/rabbitmq_config.json` with the parameters you need\


    <pre><code>{
      "rabbitmq_host": "127.0.0.1",
      "rabbitmq_password": "PqR9XJ957sfHqcxj6FsBMj4p",
      "mythic_server_host": "127.0.0.1",
      "webhook_default_channel": "#mythic-notifications",
      "debug_level": "debug",
      "rabbitmq_port": 5432,
      "mythic_server_grpc_port": 17444,
    <strong>  "webhook_default_url": "",
    </strong><strong>  "webhook_default_callback_channel": "",
    </strong>  "webhook_default_feedback_channel": "",
      "webhook_default_startup_channel": "",
      "webhook_default_alert_channel": "",
      "webhook_default_custom_channel": "",
    }
    </code></pre>

    1. the `mythic_server_host` value should be the IP address of the main Mythic install
    2. the `rabbitmq_host` value should be the IP address of the main Mythic install unless you're running rabbitmq on another host.
    3. You'll need the password of rabbitmq from your Mythic instance. You can either get this from the `Mythic/.env` file, by running `sudo ./mythic-cli config get rabbitmq_password`, or if you run `sudo ./mythic-cli config payload` you'll see it there too.
6. External agents need to connect to `mythic_rabbitmq` in order to send/receive messages. By default, this container is bound on localhost only. In order to have an external agent connect up, you will need to adjust this in the `Mythic/.env` file to have `RABBITMQ_BIND_LOCALHOST_ONLY=false` and restart Mythic (`sudo ./mythic-cli restart`). You'll also need to set `MYTHIC_SERVER_BIND_LOCALHOST_ONLY=false`.
7. In the file where you define your payload type is where you define what it means to "build" your agent.
8. Run `python3.10 main.py` and now you should see this container pop up in the UI
9. If you already had the corresponding payload type registered in the Mythic interface, you should now see the red text turn green.

You should see output similar to the following:

```
itsafeature@spooky my_container % python3 main.py    
INFO 2023-04-03 21:17:10,899 initialize  29  : [*] Using debug level: debug
INFO 2023-04-03 21:17:10,899 start_services  267 : [+] Starting Services with version v1.0.0-0.0.7 and PyPi version 0.2.0-rc9

INFO 2023-04-03 21:17:10,899 start_services  270 : [*] Processing webhook service
INFO 2023-04-03 21:17:10,899 syncWebhookData  261 : Successfully started webhook service
INFO 2023-04-03 21:17:10,899 start_services  281 : [*] Processing agent: apfell
INFO 2023-04-03 21:17:10,902 syncPayloadData  104 : [*] Processing command jsimport
INFO 2023-04-03 21:17:10,902 syncPayloadData  104 : [*] Processing command chrome_tabs
DEBUG 2023-04-03 21:17:10,915 SendRPCMessage  132 : Sending RPC message to pt_sync
INFO 2023-04-03 21:17:10,915 GetConnection  84  : [*] Trying to connect to rabbitmq at: 127.0.0.1:5672
INFO 2023-04-03 21:17:10,999 GetConnection  98  : [+] Successfully connected to rabbitmq
INFO 2023-04-03 21:17:11,038 ReceiveFromMythicDirectTopicExchange  306 : [*] started listening for messages on emit_webhook.new_callback
INFO 2023-04-03 21:17:11,038 ReceiveFromMythicDirectTopicExchange  306 : [*] started listening for messages on emit_webhook.new_feedback
INFO 2023-04-03 21:17:11,051 ReceiveFromMythicDirectTopicExchange  306 : [*] started listening for messages on emit_webhook.new_startup
INFO 2023-04-03 21:17:13,240 syncPayloadData  123 : [+] Successfully synced apfell

```

{% hint style="warning" %}
If you mythic instance has a randomized password for `rabbitmq_password`, then you need to make sure that the password from `Mythic/.env` after you start Mythic for the first time is copied over to your vm. You can either add this to your `rabbitmq_config.json` file or set it as an environment variable (`MYTHIC_RABBITMQ_PASSWORD`).
{% endhint %}

#### Caveats

There are a few caveats to this process over using the normal process. You're now responsible for making sure that the right python version and dependencies are installed, and you're now responsible for making sure that the user context everything is running from has the proper permissions.

One big caveat people tend to forget about is paths. Normal containers run on \*nix, but you might be doing this dev on Windows. So if you develop everything for windows paths hard-coded and then want to convert it to a normal Docker container later, that might come back to haunt you.

### Debugging Locally

Whether you're using a Docker container or not, you can load up the code in your `agent_code` folder in any IDE you want. When an agent is installed via `mythic-cli`, the entire agent folder (`agent_code` and `mythic`) is mapped into the Docker container. This means that any edits you make to the code is automatically reflected inside of the container without having to restart it (pretty handy). The only caveat here is if you make modifications to the python or golang definition files will require you to restart your container to load up the changes `sudo ./mythic-cli start [payload name]`. If you're making changes to those from a non-Docker instance, simply stop your `python3.8 main.py` and start it again. This effectively forces those files to be loaded up again and re-synced over to Mythic.

#### Debugging Agent Code Locally

If you're doing anything more than a typo fix, you're going to want to test the fixes/updates you've made to your code before you bother uploading it to a GitHub project, re-installing it, creating new agents, etc. Luckily, this can be super easy.

Say you have a Visual Studio project set up in your `agent_code` directory and you want to just "run" the project, complete with breakpoints and configurations so you can test. The only problem is that your local build needs to be known by Mythic in some way so that the Mythic UI can look up information about your agent, your "installed" commands, your encryption keys, etc.&#x20;

To do this, you first need to generate a payload in the Mythic UI (or via Mythic's Scripting). You'll select any C2 configuration information you need, any commands you want baked in, etc. When you click to build, all of that configuration will get sent to your payload type's "build" function in `mythic/agent_functions/builder.py`. Even if you don't have your container running or it fails to build, no worries, Mythic will first save everything off into the database before trying to actually build the agent. In the Mythic UI, now go to your payloads page and look for the payload you just tried to build. Click to view the information about the payload and you'll see a summary of all the components you selected during the build process, along with some additional pieces of information (payload UUID and generated encryption keys).&#x20;

Take that payload UUID and the rest of the configuration and stamp it into your `agent_code` build. For some agents this is as easy as modifying the values in a Makefile, for some agents this can all be set in a `config` file of some sort, but however you want to specify this information is up to you. Once all of that is set, you're free to run your agent from within your IDE of choice and you should see a callback in Mythic. At this point, you can do whatever code mods you need, re-run your code, etc.&#x20;

#### Callbacks Aplenty

Following from the previous section, if you just use the payload UUID and run your agent, you _should_ end up with a new callback each time. That can be ideal in some scenarios, but sometimes you're doing quick fixes and want to just keep tasking the same callback over and over again. To do this, simply pull the callback UUID and encryption keys from the callback information on the active callbacks page and plug that into your agent. Again, based on your agent's configuration, that could be as easy as modifying a Makefile, updating a config file, or you might have to manually comment/uncomment some lines of code. Once you're reporting back with the callback UUID instead of the payload UUID and using the right encryption keys, you can keep re-running your build without creating new callbacks each time.
