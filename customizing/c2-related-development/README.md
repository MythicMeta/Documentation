# C2 Related Development

## C2 Structure

Command and Control (C2) profiles are a little different in Mythic than you might be used to. Specifically, C2 profiles live in their own docker containers and act as a translation mechanism between whatever your special sauce C2 protocol/format/encryption is and what the back-end Mythic server understands (HTTP + JSON).&#x20;

By defining a C2 protocol specification, other payload types can register that they speak that C2 protocol as well and easily hook in without having to do back-end changes. By having C2 protocols divorced from the main Mythic server, you can create entirely new C2 protocols more easily and you can do them in whatever language you want. If you want to do all your work in Golang, C#, or some other language for the C2 protocol, go for it. It's all encapsulated in the C2's Docker container with whatever environment you desire.

Since there's so much variability possible within a C2 Docker container, there's some required structure and python files similar to how Payload Types are structured. This is covered in [C2 Profile Code](c2-profile-code/).

## C2 / Mythic Comms

Every C2 Docker container has an environment variable, `MYTHIC_ADDRESS` which points to `https://127.0.0.1:7443` by default. This information is pulled from the main `/Mythic/.env` file. So, if you change Mythic's main UI to HTTP on port 7444, then each C2 Docker container's `MYTHIC_ADDRESS` environment variable will have the value `http://127.0.0.1:7444`. This allows your code within the docker container to always know where to forward requests so that the main Mythic server can process them.

## How does a C2 Profile work in Mythic?

C2 profiles in Mythic are really straight forward - their entire role in life is to get data off the wire from whatever special communications format you're using and forward that to Mythic.&#x20;

The C2 Profile doesn't need to know anything about the actual content of the messages that are coming from agents and in most cases wouldn't be able to read them anyway since they'll be encrypted. Depending on the kind of communications you're planning on doing, your C2 Profile might wrap or break up an agent's message (eg: splitting a message to go across DNS and getting it reassembled), but then once your C2 Profile re-assembles the agent message, it can just forward it along. In most cases, simply sending the agent message as an HTTP POST message to the location specified by your container's `MYTHIC_ADDRESS` environment variable is good enough. You'll get an immediate result back from that which your C2 profile should hand back to the agent.

Ok, but what's _actually_ running? Just like with Payload Types, C2 Profiles can either run within Docker or on a separate host or within a VM somewhere. They have specific file/folder formats that are described in [c2-docker-containers.md](c2-profile-code/server-side-coding/c2-docker-containers.md "mention"). As part of that, there's the `[profile name]/c2_code` folder. This folder has two required files: `config.json` and `server`.&#x20;

* `config.json` - this is a JSON file that exposes any configuration parameters that you want to expose to the user (such as which port to open up, do you need SSL, etc).
* `server` - this is the actual piece of code that Mythic executes when you "start" a C2 Profile. This `server` file can be whatever you want as long as it's executable. For example, the `http` profile uses a Python file with `#! /usr/bin/env python3` at the top and the `websocket` profile uses a Golang binary. Whatever you plan on having this file be, just make sure it's executable and parses that `config.json` file to get its configuration.

That's it. Code up your `server` in whatever language you want, parse your `config.json` file to see what kinds of user-specified parameters exist, and you're off to the races. Everything else is just optional depending on what you're wanting to do.

## C2 RPC

C2 Profiles can access the same RPC functions that Payload Types can; however, since C2 profiles don't have things like a `task_id`, there is some functionality they won't be able to leverage. Specifically, the main functions for C2 profiles are:

* `create_event_message` - This is useful for a c2 profile to send an alert to operators in case something is weird
* `create_encrypted_message` - This is useful for a c2 profile to generate encrypted messages
* `create_decrypted_message` - This is useful for a c2 profile to decrypted messages from callbacks

These functions are available from within the C2 docker container via Python (see the http C2 `server` file as an example). These functions can be useful if you want to implement your own C2 protocol instead of JSON, but you will have to translate to/from JSON and Mythic's format at some point to interact with Mythic.&#x20;

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

