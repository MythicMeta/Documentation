# Agents 2.3 -> 3.0

With Mythic 3.0, you have two options for your containers - GoLang or Python. This section will only go over the Python side since there are no 2.3 agents with GoLang containers.&#x20;

{% hint style="warning" %}
When you git clone the new Mythic v3.0.0 you'll notice that there's no `mythic-cli` binary. To reduce the size of the GitHub clones, this binary is now included as part of the main base docker image, so run `sudo make` and the binary will be downloaded and copied into the normal spot.
{% endhint %}

The Apfell v3.0.0 ([https://github.com/MythicAgents/apfell/tree/v3.0.0](https://github.com/MythicAgents/apfell/tree/v3.0.0)) branch and the Apollo v3.0.0 ([https://github.com/MythicAgents/apollo/tree/v3.0.0](https://github.com/MythicAgents/apollo/tree/v3.0.0)) branch are good examples of two slightly different ways you can format things.

{% hint style="warning" %}
Make sure you update your Docker version to at least `20.10.22` or above (current is `23.0.1`. This is required for the latest docker containers to work properly. A simple `sudo apt upgrade` and install should suffice. Also install docker-compose via `sudo apt install docker-compose-plugin` vs the `docker-compose` script as the script will soon be deprecated according to Docker.
{% endhint %}

The `ExternalAgent` format is still the same. These next pieces are how you can test updates of your agent locally before copying your agent's folder back into your normal ExternalAgent format.

{% hint style="info" %}
The `Mythic/InstalledServices` folder is on the Mythic server. When you installed a PayloadType via `./mythic-cli install github <url>`, the folder within your github's `Payload_Type` folder is copied to the `Mythic/InstalledServices` folder.&#x20;
{% endhint %}

{% hint style="info" %}
If you are doing local agent development instead of on the same server as the Mythic instance, then in the following steps you can treat the `Mythic/InstalledServices` folder the same as your GitHub's `Payload_Type` folder (or any other folder really - it's simply serving as a staging ground while you create the new folder structure).&#x20;
{% endhint %}

{% hint style="warning" %}
If you're doing local development, you need at least Python version 3.10 because of the new typing features it offers.&#x20;
{% endhint %}

1. Make a directory, `agent name` , in `Mythic/InstalledServices`&#x20;
2. Copy your entire Payload type's `agent name` directory into `InstalledServices` (yes, the path will look like `Mythic/InstalledServices/agentName/agentName`)&#x20;
3. In `Mythic/InstalledServices/agentName` create a `main.py` and a `Dockerfile`&#x20;
4. In your new `Dockerfile`, copy the contents of your old `Dockerfile` and change the `FROM` line to `FROM itsafeaturemythic/mythic_python_base:latest`&#x20;
5.  In your new `main.py` add:

    ```
    import mythic_container
    from [agent name].mythic import *
    mythic_container.mythic_service.start_and_run_forever()
    ```


6. To make your new `agent name` directory a PyPi package that can be imported, create a `__init__.py` file in `Mythic/InstalledServices/agentName/agentName`. In the `Mythic/InstalledServices/agentName/agentName/Mythic` folder make a `__init__.py` file with the following contents (this will loop through all of your command files and import them automatically):

```python
import glob
import os.path
from pathlib import Path
from importlib import import_module, invalidate_caches
import sys
# Get file paths of all modules.

currentPath = Path(__file__)
searchPath = currentPath.parent / "agent_functions" / "*.py"
modules = glob.glob(f"{searchPath}")
invalidate_caches()
for x in modules:
    if not x.endswith("__init__.py") and x[-3:] == ".py":
        module = import_module(f"{__name__}.agent_functions." + Path(x).stem)
        for el in dir(module):
            if "__" not in el:
                globals()[el] = getattr(module, el)


sys.path.append(os.path.abspath(currentPath.name))
```

7. In your `Mythic/InstalledServices/agentName/agentName/mythic/agent_functions` files, we need to replace all `mythic_payloadtype_container` with `mythic_container` .\
   If you have an import like `from agent_functions.execute_pe import PRINTSPOOFER_FILE_ID` which references another command file, update it to `from .execute_pe import PRINTSPOOFER_FILE_ID`. If you include a local library at the same level as `agent_functions`, you can import it like `from [agent name].mythic.[package] import [thing]`

If you're doing local development, you'll need a `rabbitmq_config.json` file at the same level as your `main.py` to tell your service where Mythic is located and the rabbitmq password. The configuration options you can supply can be found in the [Local Development](../../customizing/payload-type-development/#turning-a-vm-into-a-mythic-container) section.

{% hint style="warning" %}
There are some changes to the `rabbitmq_config.json` file keys:

* `container_files_path` is no longer used and can be deleted.
* `username` is no longer used and can be deleted.
* `password` is now `rabbitmq_password.`
* `host` is now `rabbitmq_host.`
* `name` is no longer used and can be deleted.
* `virtual_host` is no longer used and can be deleted.
{% endhint %}

{% hint style="info" %}
Required keys in `rabbitmq_config.json` are:

* `rabbitmq_host` - points to the IP where Mythic lives
* `rabbitmq_password` - the password used to authenticate to rabbitmq
* `mythic_server_host` - points to the IP where Mythic lives
* `mythic_server_port` - if you're using something other than the default (this is NOT the 7443 that you use for the UI)
* mythic\_server\_grpc\_port - if you're using something other than the default
{% endhint %}

Now to actually update the content of your builder/command files. There's not much you need to do.&#x20;

#### builder.py agent definition

Because the new structure treats your entire agent directory as a Python package, the container no longer knows the paths for things. This gives you a lot more freedom in how you want to organize your code, but does require you to specify where things are located. In your `builder.py` file where you define your Payload Type, you need to add the following:

```
agent_path = pathlib.Path(".") / "apollo" / "mythic"
agent_code_path = pathlib.Path(".") / "apollo" / "agent_code"
agent_icon_path = agent_path / "agent_functions" / "apollo.svg"
```

The `agent_path` is the path to your general agent structure (typically with the `agent_functions` as a sub-folder. The `agent_code_path` points to your agent's actual code.&#x20;

Something that's a little different is the agent icons - the agents will sync that over automatically with the rest of their definition (no more having to copy it over manually or get it from an install). What that means though is you either need to supply `agent_icon_path` and provide the path to your agent's svg icon or specify `agent_icon_bytes` and provide the raw bytes for your icon.

#### build

In your payload type's build function you can report back on build steps via the `SendMythicRPCPayloadUpdateBuildStep` RPC call (based on your defined build steps). This will update the UI step-by-step for the operator so they know what's going on.

You can also set `UpdatedFilename` (or `updated_filename` for Python) in your build response and adjust the final filename of the payload. This can be helpful if your payload type allows you to build to various outputs (exe, dll, dylib, binary, etc). This allows you to adjust the filename based on that so that when the user clicks "download" in the UI, they get the right file and don't have to change the filename.

#### Browser scripts

Browserscripts work just the same, but browserscripts will look for their code at `agent_path / browser_scripts / filename.js` OR at the path specified by the `name` parameter for the script. So, that means your can either specify the name as `test.js` and have it located in your `agent_path / browser_scripts / test.js` file or specify a full path as your name.

The browser\_script attribute is a single BrowserScript value, not an array. This is because the entire Python back-end is gone, so there's no more need to supply a script for the old UI and the new UI.

#### c2 profile parameters when building

When looping through c2 profile parameters - arrays are actually arrays, crypto types and dictionary types are dictionaries, so do better checking here for name of parameters. A bunch of agents simply check if the supplied value is a dictionary and then automatically try to pull out certain values, but that might not be the case anymore. For example, when looping through the `http` profile, both the `AESPSK` and the `headers` parameters will be passed in as dictionaries.&#x20;

```python
for key, val in c2.get_parameters_dict().items():
    if key == "AESPSK":
        c2_code = c2_code.replace(key, val["enc_key"] if val["enc_key"] is not None else "")
    elif not isinstance(val, str):
        c2_code = c2_code.replace(key, json.dumps(val))
    else:
        c2_code = c2_code.replace(key, val)
```

#### Updated Create Tasking - create\_go\_tasking

The current `create_tasking` functions still work just like normal; however, the newer `create_go_tasking` function gives you more contextual data and mirrors the data structures from the new Golang container version.

```python
async def create_go_tasking(self, taskData: MythicCommandBase.PTTaskMessageAllData) -> MythicCommandBase.PTTaskCreateTaskingMessageResponse:
    response = MythicCommandBase.PTTaskCreateTaskingMessageResponse(
        TaskID=taskData.Task.ID,
        Success=True,
    )
    return response
```

This `taskData` variable is defined here: [https://github.com/MythicMeta/MythicContainerPyPi/blob/main/mythic\_container/MythicCommandBase.py#L1068](https://github.com/MythicMeta/MythicContainerPyPi/blob/main/mythic\_container/MythicCommandBase.py#L1068) and provides a lot more context in a well-defined class.

{% embed url="https://github.com/MythicMeta/MythicContainerPyPi/blob/main/mythic_container/MythicCommandBase.py#L1068" %}

#### Completion Function for Tasking

Tasks can specify for a certain function to execute when the task finishes executing. That hasn't changed. However, the format of how you define it has changed slightly. Before, you'd simply pass the name of a function and the container would loop through all known function definitions looking for one that matched. That's not super great, so now you define a dictionary of function name to function as part of your command definition.

```
completion_functions: dict[str, Callable[[PTTaskCompletionFunctionMessage], Awaitable[PTTaskCompletionFunctionMessageResponse]]] = {}
```

The PTTaskCompletionFunctionMessage and response classes can be found in the PyPi code and auto-completed via IDEs. This syntax is just the Python way of saying that the format is:\


```python
async def functionName(myArg: PTTaskCompletionFunctionMessage) -> PTTaskCompletionFunctionMessageResponse:
    do something here
```

To leverage this new `functionName` function as part of your tasking, in your `create_tasking` function you need to set the name:

```python
async def create_tasking(self, task: MythicTask) -> MythicTask:
        task.completed_callback_function = "functionName"
        return task
```

If you're using the new `create_go_tasking` function, then you need to do somthing very similar:

```python
async def create_go_tasking(self, taskData: MythicCommandBase.PTTaskMessageAllData) -> MythicCommandBase.PTTaskCreateTaskingMessageResponse:
    response = MythicCommandBase.PTTaskCreateTaskingMessageResponse(
        TaskID=taskData.Task.ID,
        CompletionFunctionName="functionName"
    )
    return response
```

#### Process response function

Sending back data via the `process_response` key within your `responses` allows you to hook into the associated command's `process_response` function within your Payload Type's container. The format of this function has changed _slightly_:

old:

```python
async def process_response(self, response: AgentResponse):
    resp = await MythicRPC().execute("update_callback", task_id=response.task.id, sleep_info=response.response)
```

new:

```python
async def process_response(self, task: PTTaskMessageAllData, response: any) -> PTTaskProcessResponseMessageResponse:
        resp = PTTaskProcessResponseMessageResponse(TaskID=task.Task.ID, Success=True)
        await MythicRPC().execute("update_callback", task_id=task.Task.ID, sleep_info=response)
        return resp
```

#### Dynamic Query Function

Similar to the completion functions, dynamic query functions look a _little_ different, but are generally still the same:

```python
dynamic_query_function: Callable[[PTRPCDynamicQueryFunctionMessage], Awaitable[PTRPCDynamicQueryFunctionMessageResponse]] = None,
```

which is to say that the function is pre-defined (one per command parameter) and looks like:

```python
async def dynamic_query_function(myArg: PTRPCDynamicQueryFunctionMessage) -> PTRPCDynamicQueryFunctionMessageResponse:
    do something
```

#### Command OPSEC

The opsec functionality has been removed from a special CommandOPSEC class and moved to the main command class itself. So, your command can have two additional functions:

```python
async def opsec_pre(self, taskData: PTTaskMessageAllData) -> PTTTaskOPSECPreTaskMessageResponse:
    response = PTTTaskOPSECPreTaskMessageResponse(
        TaskID=taskData.Task.ID, Success=True, OpsecPreBlocked=False,
        OpsecPreMessage="Not implemented, passing by default",
    )
    return response

async def opsec_post(self, taskData: PTTaskMessageAllData) -> PTTTaskOPSECPostTaskMessageResponse:
    response = PTTTaskOPSECPostTaskMessageResponse(
        TaskID=taskData.Task.ID, Success=True, OpsecPostBlocked=False,
        OpsecPostMessage="Not implemented, passing by default",
    )
    return response
```

## C2 Profiles

C2 profiles also need to be updated for Mythic 3.0.0, in an extremely similar way to Payload Types.&#x20;

1. Make a directory, `c2 name` , in `Mythic/InstalledServices`&#x20;
2. Copy your entire C2 Profile's `c2 name` directory into `InstalledServices` (yes, the path will look like `Mythic/InstalledServices/c2Name/c2Name`)&#x20;
3. Remove `c2_service.sh`, `mythic_service.py`, and `rabbitmq_config.json` from your `mythic` folder
4. Remove `C2_RPC_Functions.py`&#x20;
5. In `Mythic/InstalledServices/c2Name` create a `main.py` and a `Dockerfile`&#x20;
6. In your new `Dockerfile`, copy the contents of your old `Dockerfile` and change the `FROM` line to `FROM itsafeaturemythic/mythic_python_base:latest`&#x20;
7. In your `mythic/c2_functions/` folder, your definition file should import `mythic_container` instead of `mythic_c2_container` (similar to what we did for agent updates).
8.  In your new `main.py` add:

    ```
    import mythic_container
    from [c2 name].mythic import *
    mythic_container.mythic_service.start_and_run_forever()
    ```
9. In your c2 profile definition, add in two more attributes - `server_folder_path` (path to the folder where your server binary and config.json files exist), and `server_binary_path` (path to the binary to execute if you're doing an egress c2 profile and not a p2p profile).

To see what this looks like all together, look at the `websocket` example here: [https://github.com/MythicMeta/ExampleContainers/tree/main/Payload\_Type/python\_services](https://github.com/MythicMeta/ExampleContainers/tree/main/Payload\_Type/python\_services). You'll notice that the `websocket` is just one of _multiple_ services that the single docker contianer is offering. If you want your container to _only_ offer that one, then you can remove the other folders and adjust your `main.py` accordingly.

## Translation Containers

Translation containers are no different than C2 Profiles and Payload Types for the new format of things. Look to `translator` in the ExampleContainers ([https://github.com/MythicMeta/ExampleContainers/tree/main/Payload\_Type/python\_services](https://github.com/MythicMeta/ExampleContainers/tree/main/Payload\_Type/python\_services)) repository for an example of how to format your new structure. Translation containers boil down to one class definition with a few functions.

One big change from Mythic 2.3 -> 3.0 for Translation Containers is that they now operate over gRPC instead of RabbitMQ. This means that they need to access the gRPC port on the Mythic Server if you intend on running a translation container on a separate host from Mythic itself. This port is configurable in the `Mythic/.env` file, but by default it's 17443. This change to gRPC instead of RabbitMQ for the translation container messages speeds things up and reduces the burden on RabbitMQ for transmitting potentially large messages.
