# C2 Related Development

## What are C2 Profiles

Command and Control (C2) profiles are a little different in Mythic than you might be used to. Specifically, C2 profiles live in their own docker containers and act as a translation mechanism between whatever your special sauce C2 protocol is and what the back-end Mythic server understands (HTTP + JSON). Their entire role in life is to get data off the wire from whatever special communications format you're using and forward that to Mythic.

By defining a C2 protocol specification, other payload types can register that they speak that C2 protocol as well and easily hook in without having to do back-end changes. By having C2 protocols divorced from the main Mythic server, you can create entirely new C2 protocols more easily and you can do them in whatever language you want. If you want to do all your work in Golang, C#, or some other language for the C2 protocol, go for it. It's all encapsulated in the C2's Docker container with whatever environment you desire.

Since there's so much variability possible within a C2 Docker container, there's some required structure and python files similar to how Payload Types are structured. This is covered in [C2 Profile Code](c2-profile-code/).

## How does a C2 Profile work in Mythic?

When we look at how C2 Profiles work within Mythic, there are two different stages to consider:

1. How is the C2 Profile defined so that Mythic can track all of the parameters and present them to the user when generating payloads.
2. How does the C2 Profile's code run so that it can listen for agent traffic and communicate with Mythic.

#### Step 1 - Defining your Profile

Just like with Payload Types, C2 Profiles can either run within Docker or on a separate host or within a VM somewhere. This isn't a hard requirement, but makes it easier to share them. As part of this, there's a standard file and folder format that's used so that Mythic can properly process all of the components and start the right files. This format is described in [c2-docker-containers.md](c2-profile-code/server-side-coding/c2-docker-containers.md "mention"). As part of this, you can copy the sample `Example_C2_Profile` folder from Mythic. Copy this folder to the `Mythic/C2_Profiles/` folder and rename it to the same name as the C2 profile you want to create. Inside of this [folder](https://github.com/its-a-feature/Mythic/tree/master/Example\_C2\_Profile) are a few important pieces:

{% hint style="info" %}
If you're going to be using the `mythic-cli` to install and run your C2 Profile, then Mythic will mount your `Mythic/C2_Profiles/[c2 profile name]` folder as `/Mythic` inside of the Docker container as a volume. This means that any changes to the `Mythic/C2_Profiles/[c2 profile name]` folder that happen on disk will be mirrored inside of the Docker container.
{% endhint %}

* Dockerfile - this is the Dockerfile for your C2 profile that defines the environment and what's installed
* c2\_code - this folder is what will hold the server code and configuration files for your C2 profile.
  * In this folder, the two required files are `config.json` and `server`
  * `config.json` - this is a JSON file that exposes any configuration parameters that you want to expose to the user (such as which port to open up, do you need SSL, etc).
  * `server` - this is the actual piece of code that Mythic executes when you "start" a C2 Profile. This `server` file can be whatever you want as long as it's executable. For example, the `http` profile uses a Python file with `#! /usr/bin/env python3` at the top, and the `websocket` profile uses a Golang binary. Whatever you plan on having this file be, just make sure it's executable and parses that `config.json` file to get its configuration.
* mythic - this folder holds the components that are Mythic related. This includes definition files for your profile (what kind of parameters does it take, what's the name, etc) as well as your OPSEC checks and potential redirector generation code.
  * In this [folder](https://github.com/its-a-feature/Mythic/tree/master/Example\_C2\_Profile/mythic), you'll see a few important files and folders.&#x20;
  * `c2_functions` - this holds the actual Python definition files and potential RPC functionality you want to expose.
    * Inside of this [folder](https://github.com/its-a-feature/Mythic/tree/master/Example\_C2\_Profile/mythic/c2\_functions), you'll see two main files. `C2_RPC_Functions.py` is where you'll define the RPC functions and OPSEC checking code you'll expose to the rest of Mythic. The other file (by default just called `HTTP.py`) is what holds all of the important definition information for your profile. It's in this file that you will set the actual name for your profile and what parameters it takes.
  * `c2_service.sh` - this is the entrypoint for the docker container. This changes the current working directory and gets everything set up to then call the `mythic_service.py` file.
  * `mythic_service.py` - this python file is what acts as the long running service in the Docker container which connects up to Mythic, sends heartbeats, and processes messages from Mythic (such as doing OPSEC checks, starting/stopping the profile, updating the config, etc).
  * `rabbitmq_config.json` - this configuration file indicates how the `mythic_service.py` file will connect to RabbitMQ so that it can get/send messages with Mythic. This will include things like the IP address and password for connecting as well as the name of your profile.

{% hint style="warning" %}
Make sure that `name` specified in the `rabbitmq_config.json` file matches the name you set in your definition file (within the `c2_functions` folder) exactly!&#x20;
{% endhint %}

Once you get all of that copied over and modified, you'll want to register the new C2 Profile with Mythic. Normally you install C2 Profiles from the web with `sudo ./mythic-cli install github https://github.com/C2Profiles/[profile name]`. However, since you already have the code and folder structure in your `Mythic/C2_Profiles` folder, we can just 'tell' Mythic that it exists. You can do this via `sudo ./mythic-cli c2 add [profile name]`.  You can then start just that one container with `sudo ./mythic-cli c2 start [profile name]`.  When the container starts, a few things happen:

1. The Docker container kicks off `c2_service.sh`
2. `c2_service.sh` sets up some environment variables and sets the current working directory, then kicks off `mythic_service.py`
3. `mythic_service.py` then processes the `rabbitmq_config.json` as well as environment variables passed in. It then processes all of the files within the `c2_functions` folder to look for your C2 Profile class (You'll notice [here](https://github.com/its-a-feature/Mythic/blob/master/Example\_C2\_Profile/mythic/c2\_functions/HTTP.py#L4) that your class extends the C2Profile class). Once it finds that class, it gets a dictionary representation of all of that information (C2 profile name, parameters, etc) and then connects to RabbitMQ to send that data to Mythic.&#x20;
4. When Mythic gets that synchronization message from the container with all of the dictionary information, it ties to import the C2 Profile. If it is able to successfully import (or update the current instance), then it'll report an event message that the C2 profile successfully synced.
5. Once the sync happens, the Docker container sends periodic heartbeat messages to Mythic to let it know that the container is still up and going. This is how the UI is able to determine if the container is up or down.

{% hint style="warning" %}
The heartbeat messages use the name of the container when sending their messages. Mythic will automatically set the hostname of the container to the same name as the C2 profile. This information comes from that `rabbitmq_config.json` file with the `name` parameter. This is why it's important for that to match the name of the profile _exactly_. If something doesn't match up, then you'll run into an instance where you get a heartbeat for C2 Profile X, which Mythic doesn't know. It'll then ask the container to sync up, and the container will send information about C2 Profile Y (based on the information in the `c2_functions` folder). That information will get processed, but then the container will send a heartbeat for C2 Profile X again. Again, Mythic doesn't know C2 Profile X, only Y, so it'll ask the container to sync again. And that just repeats forever.
{% endhint %}

#### Step 2 - Processing messages

The C2 Profile doesn't need to know anything about the actual content of the messages that are coming from agents and in most cases wouldn't be able to read them anyway since they'll be encrypted. Depending on the kind of communications you're planning on doing, your C2 Profile might wrap or break up an agent's message (eg: splitting a message to go across DNS and getting it reassembled), but then once your C2 Profile re-assembles the agent message, it can just forward it along. In most cases, simply sending the agent message as an HTTP POST message to the location specified by your container's `MYTHIC_ADDRESS` environment variable is good enough. You'll get an immediate result back from that which your C2 profile should hand back to the agent.

Mythic will try to automatically start your `[c2 profile name]/c2_code/server` file when the container starts. This same file is what gets executed when you click to "start" the profile in the UI.&#x20;

Every C2 Docker container has an environment variable, `MYTHIC_ADDRESS` which points to `https://127.0.0.1:7443` by default. This information is pulled from the main `/Mythic/.env` file. So, if you change Mythic's main UI to HTTP on port 7444, then each C2 Docker container's `MYTHIC_ADDRESS` environment variable will have the value `http://127.0.0.1:7444`. This allows your code within the docker container to always know where to forward requests so that the main Mythic server can process them.

{% hint style="info" %}
The C2 Profile has nothing to do with the _content_ of the messages that are being sent. It has no influence on the encryption or what format the agent messages are in (JSON, binary, stego, etc). If you want to control that level of granularity, you need to check out the [translation-containers.md](../payload-type-development/translation-containers.md "mention").&#x20;
{% endhint %}

{% hint style="info" %}
When forwarding messages to Mythic, they must be in a specific format: Base64(UUID + message). This just allows Mythic to have a standard way to process messages that are coming in and pull out the needed pieces of information. The UUID allows mythic to look up the associated Payload Type and see what needs to happen (is it a payload that's staging, is it a callback, does processing need to go to a translation container first, etc).The `message` is typically an encrypted blob, but could be anything.
{% endhint %}

## C2 RPC

C2 Profiles can access the same RPC functions that Payload Types can; however, since C2 profiles don't have things like a `task_id`, there is some functionality they won't be able to leverage. Specifically, the main functions for C2 profiles are:

* `create_event_message` - This is useful for a c2 profile to send an alert to operators in case something is weird
* `create_encrypted_message` - This is useful for a c2 profile to generate encrypted messages
* `create_decrypted_message` - This is useful for a c2 profile to decrypted messages from callbacks

These functions are available from within the C2 docker container via Python (see the http C2 `server` file as an example). These functions can be useful if you want to implement your own C2 protocol instead of JSON, but you will have to translate to/from JSON and Mythic's format at some point to interact with Mythic.

### Payload Type Docker -> C2 Docker

This one is a little less intuitive than the C2 Docker container directly reaching out to the Mythic server for functionality. This functionality allows tasking as an operator to directly manipulate a C2 component. This functionality has no "default" functions, it's all based on the C2 profile itself. Here is an example of a simple RPC function from a Payload Type Docker container -> C2 Docker container:

```python
from mythic_c2_container.MythicRPC import *

# request is a dictionary: {"action": func_name, "message": "the input",  "task_id": task id num}
# must return an RPCResponse() object and set .status to an instance of RPCStatus and response to str of message
async def test(request):
    response = RPCResponse()
    response.status = RPCStatus.Success
    response.response = "hello"
    resp = await MythicRPC().execute("create_event_message", message="Test message", warning=False)
    return response
```

Within the C2 profile's `mythic/c2_functions` folder can be any number of python files that have defined functions. These functions are automatically imported and available for RPC when the container starts. In this case, a remote container can call this C2's `test` function.
