---
description: This section describes new Payload Types
---

# Payload Type Development

## Creating a new Mythic agent

You want to create a new agent that fully integrates with Mythic. Since everything in Mythic revolves around Docker containers, you will need to ultimately create one for your payload type along with some specific files/folder structures. This can be done with docker containers on the same host as the Mythic server or with an [external VM/host machine](./#turning-a-vm-into-an-apfell-container).&#x20;

* Optionally: You can also have a custom C2 profile that this agent supports, see [C2 Profile Development](../c2-related-development/) for that piece though

{% hint style="info" %}
The first step is to look into the [First Steps](first-steps.md) page to get everything generally squared away and look at the order of things that need your attention. The goal is to get you up and going as fast as possible!
{% endhint %}

## Payload/C2 Development Resources

There are a lot of things involved in making your own agent, c2 profile, or translation container. For the bare-bones implementations, you'll have to do very little, but Mythic tries to allow a lot of customizability. To help with this, Mythic uses PyPi packages for a lot of its configuration/connections and Docker images for distribution. This makes it easy for somebody to just kind of "hook up" and get going. If you don't want to do that though, all of this code is available to you within the MythicMeta organization on GitHub ([https://github.com/MythicMeta](https://github.com/MythicMeta)).&#x20;

This organization is broken out in five main repos:

* `Mythic_CLI` - This holds all of the code for the PyPi package, `mythic`, that you can use to script up actions.
* `Mythic_Translator_Container` - This holds all of the code for the PyPi package, `mythic_translator_container`, that you can use to build your own translation container.&#x20;
* `Mythic_PayloadType_Container` - This holds all of the code for the PyPi package, `mythic_payloadtype_container`, that you can use to create your own payload type docker image or when turning a vm into your own payloadtype container.
* `Mythic_C2Profile_Container` - This holds all of the code for the PyPi package, `mythic_c2_container`, that you can use to create your own c2 profile docker image or when turning an arbitrary host into a c2 profile service.
* `Mythic_DockerTemplates` - This holds all of the code and resources that are used to make all of the Docker images hosted on DockerHub ([https://hub.docker.com/search?q=itsafeaturemythic\&type=image](https://hub.docker.com/search?q=itsafeaturemythic\&type=image)). This is helpful if you want to see what's actually happening for a specific container or you want to use one of these as a starting point for your own containers.

## Payload Type Docker Information

The rest of these sections talk about specifics for the docker container for your new agent.

### Creating a docker container for your new agent

There are docker containers for each payload type which contain customized build environments. This allows two payload types that might share the same language to still have different environment variables, build paths, tools installed, etc. Docker containers come into play for a few things:

* Metadata about the payload type (this is in the form of python classes)
* The payload type code base (whatever language your agent is in)
* The python code to create the payload based on all of the user supplied input
* Metadata about all of the commands associated with that payload type (more python classes)
* The code for all of those commands (whatever language your agent is in)
* Browser scripts for commands and support scripts for the payload type as a whole (JavaScript)
* The python code to take user supplied tasking and turn it into tasking for your agent

{% hint style="warning" %}
This part has to happen outside of the web UI at the moment. The web UI is running in its own docker container, and as such, can't create, start, or stop entire docker containers. So, make sure this part is done via CLI access to where Mythic is installed.
{% endhint %}

Within `Mythic/Payload_Types/` make a new folder that matches the name of your agent. Inside of this folder make a file called `Dockerfile`. This is where you have a choice to make - either use the default Payload Type docker container as a starting point and make your additions from there, or use a different container base.

#### Using the default container base

The default container is pretty bare bones except for python 3.8. Start your `Dockerfile` off with:

```
From itsafeaturemythic/python38_payload:0.0.7
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

There are a few different default containers you can leverage depending on the environment you need:

* From itsafeaturemythic/python38\_payload:0.0.7
  * This is based on `python:3.8-buster` and only has python3.8 installed
* From itsafeaturemythic/csharp\_payload:0.0.13
  * This is based on `mono:latest` with python 3.8.6 manually installed along with the `System.Management.Automation.dll` added in (v2 and v4)
*   From itsafeaturemythic/xgolang\_payload:0.0.12

    * This is based on `karalabe/xgo-latest` with python 3.8.6 manually installed



The latest container versions and their associated `mythic_payloadtype_container` PyPi versions can be found [here](container-syncing.md#current-payloadtype-versions).

If you're curious what else goes into these containers, look in the following repo:

{% embed url="https://github.com/MythicMeta/Mythic_Docker_Templates/tree/master/Docker_Payload_Type_base_files" %}

#### Required Folder Structure

The `Mythic/Payload_Types/[agent name]` folder is mapped to `/Mythic` in the docker container. Editing the files on disk results in the edits appearing in the docker container and visa versa.

All of your agent code, commands, and c2 profile code should be in the following folder:\
`Mythic/Payload_Types/[agent name]/agent_code/`. You can have any folder structure or files you want here.&#x20;

The `Mythic/Payload_Types/[agent name]/mythic`  folder contains all information for interacting with Mythic. Inside of the `mythic` folder there's a subfolder `agent_functions` where all of your agent-specific building/command information lives.&#x20;

{% hint style="info" %}
The entire folder structure can be copied from the `Example_Payload_Type` folder.
{% endhint %}

There isn't _too_ much that's different if you're going to use your own container, it's just on you to make sure that python3.8 is up and running and the entrypoint is set properly. Here's what the base containers do:

{% embed url="https://github.com/MythicMeta/Mythic_Docker_Templates/blob/master/Docker_Payload_Type_base_files/python38_dockerfile" %}

with the following requirements.txt file in them:

{% embed url="https://github.com/MythicMeta/Mythic_Docker_Templates/blob/master/Docker_Payload_Type_base_files/requirements.txt" %}

Notice that it installs python3.8, sets it up correctly, installs the required packages (aio\_pika, mythic-payloadtype-container, dynaconf) for Mythic, and sets the entrypoint for a default service file.&#x20;

{% hint style="info" %}
If you're having this hook up through Mythic via `mythic-cli` and the one `docker-compose` file, the `dynaconf` and `mythic-payloadtype-container` are the ones responsible for the `Mythic/.env` configuration being applied to your container.&#x20;
{% endhint %}

{% hint style="warning" %}
If your container/service is running on a different host than the main Mythic instance, then you need to make sure the `rabbitmq_password` is shared over to your agent as well. By default, this is a randomized value stored in the `Mythic/.env` file and shared across containers, but you will need to manually share this over with your agent either via an environment variable (`MYTHIC_RABBITMQ_PASSWORD` or by editing the `rabbitmq_password` field in your rabbitmq\_config.json file.
{% endhint %}

### Starting your Docker container

To start your new payload type docker container, you need to first make sure that the docker-compose file is aware of it (assuming you didn't install it via `mythic-cli install github <url>` ). You can check if your `docker-compose` file is aware of your agent via `mythic-cli payload list`. If it's not aware, youc an simply do `mythic-cli payload add <payload name>`. Now you can start just that one container via `mythic-cli payload start <payload name>`.

Your container should pull down all the necessary files, run the necessary scripts, and finally start. If it started successfully, you should see it listed in the payload types section when you run `sudo ./mythic-cli status`.&#x20;

If you go back in the web UI at this point, you should see the red light next to your payload type change to green to indicate that it's now getting heartbeats. If it's not, then something went wrong along the way. You can use the `sudo ./mythic-cli logs payload_type_name` to see the output from the container to potentially troubleshoot.

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

So, to leverage your own custom VM or physical computer into a Mythic recognized container, there are just a few steps.&#x20;

1. Install python 3.8+ in the VM
2. pip3 install aio\_pika (this is used to communicate via rabbitmq)
3. pip3 install mythic-payloadtype-container (this has all of the definitions and functions for the container to sync with Mythic and issue RPC commands)
4. Create a folder on the computer or VM (let's call it path `/pathA`).
5. Do the same folder structure and files as above in the Payload_Types/\[agent  name] folder (_copy everything from the `Example_Payload_Type`_ _folder). Essentially, your `/pathA` path will be the new `Payload_Types/[agent name]` folder.
6. Edit the `rabbitmq_config.json` with the parameters you need
   1. the `host` value should be the IP address of the main Mythic install
   2. the `name` value should be the name of the payload type (this is tied into how the routing is done within rabbitmq). For Mythic's normal docker containers, this is set to `hostname` because the hostname of the docker container is set to the name of the payload type. For this case though, that might not be true. So, you can set this value to the name of your payload type instead.
   3. the `container_files_path` should be the absolute path to the folder in step 3 (`/pathA`in this case)
7. export a `PYTHONPATH` variable adding your `/pathA` and `/pathA/mythic`
8. Run  `python3 mythic_service.py` and now you should see this container pop up in the UI
9. If you already had the corresponding payload type registered in the Mythic interface, you should now see the red light turn green.&#x20;

{% hint style="warning" %}
If you mythic instance has a randomized password for `rabbitmq_password`, then you need to make sure that the password from `Mythic/.env` after you start Mythic for the first time is copied over to your vm. You can either add this to your `rabbitmq_config.json` file or set it as an environment variable (`MYTHIC_RABBITMQ_PASSWORD`).&#x20;
{% endhint %}

### Caveats

There are a few caveats to this process over using the normal process. You're now responsible for making sure that the right python version and dependencies are installed, and you're now responsible for making sure that the user context everything is running from has the proper permissions.

One big caveat people tend to forget about is paths. Normal containers run on \*nix, but you might be doing this dev on Windows. So if you develop everything for windows paths hard-coded and then want to convert it to a normal Docker container later, that might come back to haunt you.
