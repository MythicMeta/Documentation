---
description: What are the first things to do when creating a new payload type in Mythic?
---

# First Steps

## So you want to make a new Payload Type

The first step is to copy the `/Mythic/Example_Payload_Type` folder into the `/Mythic/Payload_Types/` folder and rename it to match the name of your new agent. For the purposes here, let's assume your new agent is called `my_agent`. So you'd copy that directory as `/Mythic/Payload_Types/my_agent`.

{% hint style="warning" %}
Docker does not allow capital letters in container names. So, if you plan on using Mythic `mythic-cli` to control and install your agent, then your agent's name can't have any capital letters in it. Only lowercase, numbers, and \_. It's a silly limitation by Docker, but it's what we're working with.
{% endhint %}

The next thing you need to do is edit the [builder](https://github.com/its-a-feature/Mythic/blob/master/Example\_Payload\_Type/mythic/agent\_functions/builder.py) file ([https://github.com/its-a-feature/Mythic/blob/master/Example\_Payload\_Type/mythic/agent\_functions/builder.py](https://github.com/its-a-feature/Mythic/blob/master/Example\_Payload\_Type/mythic/agent\_functions/builder.py)) to reflect the information for your new agent. Specifically, you'll want to edit it like:

```python
from mythic_payloadtype_container.MythicCommandBase import *
from mythic_payloadtype_container.PayloadBuilder import *
import asyncio
import os
from distutils.dir_util import copy_tree
import tempfile

# define your payload type class here, it must extend the PayloadType class though
class MyAgent(PayloadType):

    name = "my_agent"  # name that would show up in the UI
    file_extension = "exe"  # default file extension to use when creating payloads
    author = "@YourHandleHere"  # author of the payload type
    supported_os = [  # supported OS and architecture combos
        SupportedOS.Windows, SupportedOS.Linux # update this list with all the OSes your agent supports
    ]
    wrapper = False  # does this payload type act as a wrapper for another payloads inside of it?
    # if the payload supports any wrapper payloads, list those here
    wrapped_payloads = [] # ex: "service_wrapper"
    note = "Any note you want to show up about your payload type in the UI"
    supports_dynamic_loading = False  # setting this to True allows users to only select a subset of commands when generating a payload
    build_parameters = [
        #  these are all the build parameters that will be presented to the user when creating your payload
        # we'll leave this blank for now
    ]
    #  the names of the c2 profiles that your agent supports
    c2_profiles = ["http"]
    # after your class has been instantiated by the mythic_service in this docker container and all required build parameters have values
    # then this function is called to actually build the payload
    async def build(self) -> BuildResponse:
        # this function gets called to create an instance of your payload
        resp = BuildResponse(status=BuildStatus.Error)
        return resp
```

More information on each component in the file can be found in [Payload Type Info](payload-type-info.md). Now you can run `sudo ./mythic-cli payload add my_agent` and `sudo ./mythic-cli payload start MyNewAgent` and you'll see the container build, start, and you'll see it sync with the Mythic server (more about that process at [Container Syncing](container-syncing.md)).

{% hint style="success" %}
Congratulations! You now have a payload type that Mythic recognizes!
{% endhint %}

Now you'll want to actually configure your [Docker Container](./#payload-type-docker-information), look into [building your agent](payload-type-info.md), how to declare [new commands](commands.md#commandbase), how to [process tasking](create\_tasking.md) to these commands, and finally [hooking your agent](../hooking-features/) into all the cool features of Mythic.
