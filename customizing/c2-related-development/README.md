# 2. C2 Development

## What are C2 Profiles

Command and Control (C2) profiles are a little different in Mythic than you might be used to. Specifically, C2 profiles live in their own docker containers and act as a translation mechanism between whatever your special sauce C2 protocol is and what the back-end Mythic server understands (HTTP + JSON). Their entire role in life is to get data off the wire from whatever special communications format you're using and forward that to Mythic.

By defining a C2 protocol specification, other payload types can register that they speak that C2 protocol as well and easily hook in without having to do back-end changes. By having C2 protocols divorced from the main Mythic server, you can create entirely new C2 protocols more easily and you can do them in whatever language you want. If you want to do all your work in GoLang, C#, or some other language for the C2 protocol, go for it. It's all encapsulated in the C2's Docker container with whatever environment you desire.

Since there's so much variability possible within a C2 Docker container, there's some required structure and python files similar to how Payload Types are structured. This is covered in [C2 Profile Code](broken-reference).

## How does a C2 Profile work in Mythic?

When we look at how C2 Profiles work within Mythic, there are two different stages to consider:

1. How is the C2 Profile defined so that Mythic can track all of the parameters and present them to the user when generating payloads.
2. How does the C2 Profile's code run so that it can listen for agent traffic and communicate with Mythic.

#### Step 1 - Defining your Profile

Just like with Payload Types, C2 Profiles can either run within Docker or on a separate host or within a VM somewhere. This isn't a hard requirement, but makes it easier to share them. The format is the same as for Payload Types (and even Translation Containers) - the only difference is which classes/structs we instantiate. Check out [payload-type-development](../payload-type-development/ "mention") for the general structure.

{% hint style="info" %}
If you're going to be using the `mythic-cli` to install and run your C2 Profile, then Mythic will mount your `Mythic/InstalledServices/[c2 profile name]` folder as `/Mythic` inside of the Docker container as a volume. This means that any changes to the `Mythic/InstalledServices/[c2 profile name]` folder that happen on disk will be mirrored inside of the Docker container.
{% endhint %}

Some differences to note:

* Just like how Payload Types have two sections (agent code and Mythic definition files), C2 Profiles have the same sort of thing (agent code and Mythic definition files).
  * Wherever your server code is located, there's a required file, `config.json`, that the user can edit from the Mythic UI.&#x20;
    * `config.json` - this is a JSON file that exposes any configuration parameters that you want to expose to the user (such as which port to open up, do you need SSL, etc).
    *   ```
        server_binary_path = pathlib.Path(".") / "websocket" / "c2_code" / "server"
        ```

        `server_binary_path` - this is the actual program that Mythic executes when you "start" a C2 Profile. This file can be whatever you want as long as it's executable.
  * The Mythic definition files for your profile (what kind of parameters does it take, what's the name, etc) as well as your OPSEC checks and potential redirector generation code.

Once you get all of that created, you'll want to register the new C2 Profile with Mythic. Normally you install C2 Profiles from the web with `sudo ./mythic-cli install github https://github.com/C2Profiles/[profile name]`. However, since you already have the code and folder structure in your `Mythic/InstalledServices` folder, we can just 'tell' Mythic that it exists. You can do this via `sudo ./mythic-cli add [profile name]`.  You can then start just that one container with `sudo ./mythic-cli start [profile name]`.  When the container starts, a few things happen:

1. The Docker container kicks off `main.py` or `main` depending on Python or GoLang
2. The optional `rabbitmq_config.json` as well as environment variables passed in are processed and used to start the service. It then processes all of the files within the `c2_functions` folder to look for your C2 Profile class (You'll notice [here](https://github.com/its-a-feature/Mythic/blob/master/Example\_C2\_Profile/mythic/c2\_functions/HTTP.py#L4) that your class extends the C2Profile class). Once it finds that class, it gets a dictionary representation of all of that information (C2 profile name, parameters, etc) and then connects to RabbitMQ to send that data to Mythic.&#x20;
3. When Mythic gets that synchronization message from the container with all of the dictionary information, it ties to import the C2 Profile. If it is able to successfully import (or update the current instance), then it'll report an event message that the C2 profile successfully synced.
4. Once the sync happens, the Docker container sends periodic heartbeat messages to Mythic to let it know that the container is still up and going. This is how the UI is able to determine if the container is up or down.

#### Step 2 - Processing messages

The C2 Profile doesn't need to know anything about the actual content of the messages that are coming from agents and in most cases wouldn't be able to read them anyway since they'll be encrypted. Depending on the kind of communications you're planning on doing, your C2 Profile might wrap or break up an agent's message (eg: splitting a message to go across DNS and getting it reassembled), but then once your C2 Profile re-assembles the agent message, it can just forward it along. In most cases, simply sending the agent message as an HTTP POST message to the location specified by your container's `http://MythicServerHost:MythicServerPort/agent_message` endpoint where `MythicServerHost` and `MythicServerPort` are both available via environment variable is good enough. You'll get an immediate result back from that which your C2 profile should hand back to the agent.

Mythic will try to automatically start your server file when the container starts. This same file is what gets executed when you click to "start" the profile in the UI.&#x20;

Every Docker container has environment variables, `MYTHIC_SERVER_HOST` which points to `127.0.0.1` by default and `MYTHIC_SERVER_PORT` which points to `17443` by default. This information is pulled from the main `/Mythic/.env` file. So, if you change Mythic's main UI to HTTP on port 7444, then each C2 Docker container's `MYTHIC_SERVER_PORT` environment variable will update. This allows your code within the docker container to always know where to forward requests so that the main Mythic server can process them.

{% hint style="info" %}
The C2 Profile has nothing to do with the _content_ of the messages that are being sent. It has no influence on the encryption or what format the agent messages are in (JSON, binary, stego, etc). If you want to control that level of granularity, you need to check out the [translation-containers.md](../payload-type-development/translation-containers.md "mention").&#x20;
{% endhint %}

{% hint style="info" %}
When forwarding messages to Mythic, they must be in a specific format: Base64(UUID + message). This just allows Mythic to have a standard way to process messages that are coming in and pull out the needed pieces of information. The UUID allows mythic to look up the associated Payload Type and see what needs to happen (is it a payload that's staging, is it a callback, does processing need to go to a translation container first, etc).The `message` is typically an encrypted blob, but could be anything.
{% endhint %}

## C2 RPC

C2 Profiles can access the same RPC functions that Payload Types can; however, since C2 profiles don't have things like a `task_id`, there is some functionality they won't be able to leverage.&#x20;

### Payload Type Docker -> C2 Docker

This one is a little less intuitive than the C2 Docker container directly reaching out to the Mythic server for functionality. This functionality allows tasking as an operator to directly manipulate a C2 component. This functionality has no "default" functions, it's all based on the C2 profile itself. Technically, this goes both ways - C2 Profiles can reach back and execute functionality from Payload Types as well.

Payload Types and C2 Profiles can specify an attribute, `custom_rpc_functions`, which are dictionaries of `key`-`value` pairs (much like the completion functions) where the `key` is the name of the function that a remote services can call, and the `value` is the actual function itself. These functions have the following format:

```python
async def func_name(incomingMsg: PayloadBuilder.PTOtherServiceRPCMessage) -> PayloadBuilder.PTOtherServiceRPCMessageResponse:
    response = PayloadBuilder.PTOtherServiceRPCMessageResponse(
        Success=True,
        Result={"some dictionary": "with some values", **incomingMsg.ServiceRPCFunctionArguments}
    )
```

The incoming data is a dictionary in the `incomingMsg.ServiceRPCFunctionArguments` and the resulting data goes back through the `Result` key.
