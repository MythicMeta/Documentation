---
description: What are the first things to do when creating a new payload type in Mythic?
---

# First Steps

## So you want to make a new Payload Type

The first step is to copy down the example repository [https://github.com/MythicMeta/ExampleContainers](https://github.com/MythicMeta/ExampleContainers).&#x20;

Inside of the `Payload_Type` folder, there are two projects - one for GoLang and one for Python depending on which language you prefer to code your agent definitions in (this has nothing to do with the language of your agent itself, it's simply the language to define commands and parameters). Pick whichever service you're interested in and copy that folder into your `Mythic/InstalledServices` folder. If you want your new container to be referenced as `my_agent` via `mythic-cli`, then copy the folder over to `Mythic/InstalledServices/my_agent` (**note**: this is simply how `mythic-cli` will reference your container).

{% hint style="warning" %}
Docker does not allow capital letters in container names. So, if you plan on using Mythic's `mythic-cli` to control and install your agent, then your agent's name can't have any capital letters in it. Only lowercase, numbers, and \_. It's a silly limitation by Docker, but it's what we're working with.
{% endhint %}

The example services has a single container that offers multiple options (Payload Type, C2 Profile, Translation Container, Webhook, and Logging). While a single container can have all of that, for now we're going to focus on just the payload type piece, so delete the rest of it. For the `python_services` folder this would mean deleting the `mywebhook`, `translator`, and `websocket` folders. For the `go_services` folder, this would mean deleting the `http`, `my_logger`, `my_webhooks`, `no_actual_translation` folders. For both cases, this will result in removing some imports at the top of the remaining `main.py` and `main.go` files.

{% tabs %}
{% tab title="Python" %}
For the `python_services` folder, we'll update the `apfell/agent_functions/builder.py` file. This file can technically be anywhere that `main.py` can reach and import, but for convenience it's in a folder, `agent_functions` along with all of the command definitions for the agent. Below is an example from that builder that defines the agent:

```python
import logging
import pathlib
from mythic_container.PayloadBuilder import *
from mythic_container.MythicCommandBase import *
from mythic_container.MythicRPC import *
import json


class Apfell(PayloadType):
    name = "apfell"
    file_extension = "js"
    author = "@its_a_feature_"
    supported_os = [SupportedOS.MacOS]
    wrapper = False
    wrapped_payloads = []
    note = """This payload uses JavaScript for Automation (JXA) for execution on macOS boxes."""
    supports_dynamic_loading = True
    c2_profiles = ["http", "dynamichttp"]
    mythic_encrypts = True
    translation_container = None # "myPythonTranslation"
    build_parameters = []
    agent_path = pathlib.Path(".") / "apfell"
    agent_icon_path = agent_path / "agent_functions" / "apfell.svg"
    agent_code_path = agent_path / "agent_code"

    build_steps = [
        BuildStep(step_name="Gathering Files", step_description="Making sure all commands have backing files on disk"),
        BuildStep(step_name="Configuring", step_description="Stamping in configuration values")
    ]

    async def build(self) -> BuildResponse:
        # this function gets called to create an instance of your payload
        resp = BuildResponse(status=BuildStatus.Success)
        return resp
```
{% endtab %}

{% tab title="Golang" %}

{% endtab %}
{% endtabs %}

More information on each component in the file can be found in [Payload Type Info](../payload-type-info.md). Now you can run `sudo ./mythic-cli add my_agent` and `sudo ./mythic-cli build my_agent` and you'll see the container build, start, and you'll see it sync with the Mythic server (more about that process at [Container Syncing](container-syncing.md)).

**Note:** We're only doing `add` and `start` subcommands because we're not using the `install` command to install a configured repo from git or a local folder. You also only need to do the `add` once because the entire folder is mounted into the created docker container, so your changes are automatically represented within the container. As we make changes, you will have to run `sudo ./mythic-cli build my_agent` though to rebuild the container and pull in your changes.

{% hint style="success" %}
Congratulations! You now have a payload type that Mythic recognizes!
{% endhint %}

Now you'll want to actually configure your [Docker Container](../#payload-type-docker-information), look into [building your agent](../payload-type-info.md), how to declare [new commands](../commands.md#commandbase), how to [process tasking](../create\_tasking.md) to these commands, and finally [hooking your agent](../../hooking-features/) into all the cool features of Mythic.&#x20;
