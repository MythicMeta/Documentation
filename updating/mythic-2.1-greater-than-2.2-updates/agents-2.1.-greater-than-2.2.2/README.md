# Agents 2.1.\* -> 2.2.8

## Collapsed Files

The first big update for the payload type containers is to make things easier to update. To do this, many of the files within `agentName/mythic` are now in a PyPi package (`mythic-payloadtype-container`) hosted at ([https://github.com/MythicMeta/Mythic\_PayloadType\_Container](https://github.com/MythicMeta/Mythic\_PayloadType\_Container)). Specifically, your `agentName/mythic` folder will now look like:

* agent\_functions/
* browser\_scripts/
* mythic\_service.py
* payload\_service.sh
* rabbitmq\_config.json

All of those other files can be deleted.

* If you are using a DockerImage from the `itsafeaturemythic` repo, update to the latest:
  * csharp\_payload==0.0.14
  * python38\_payload==0.0.7
  * xgolang\_payload==0.0.12
  * leviathan\_payload==0.0.7
* If you're rolling your own, make sure you use the `mythic_payloadtype_container==0.0.45` PyPi package.

### mythic\_service.py

This is the main file that kicks off and runs all of the container code for sending heartbeats and interacting with Mythic. This is now a super simple script:

```python
#!/usr/bin/env python3
from mythic_payloadtype_container import mythic_service
mythic_service.start_service_and_heartbeat(debug=False)
```

If you want more debugging information from this container, simply set `debug=True`.

### payload\_service.sh

This file just changes to the right directory inside of the Docker container, makes sure that the new Mythic directory is in your Python environment path, and kicks off that service file we just covered:

```
#!/bin/bash
cd /Mythic/mythic
export PYTHONPATH=/Mythic:/Mythic/mythic
python3.8 mythic_service.py
```

## Dockerfile

The Docker files and Docker images for Mythic agents have all been updated. Instead of housing all of this information inside of the main Mythic repo, these have all been moved to the MythicMeta organization under [Mythic\_Docker\_Templates](https://github.com/MythicMeta/Mythic\_Docker\_Templates). They all now include a python `requirements.txt` file so it's easy to see what software is needed if you want to create your own Docker image or turn a VM into the Mythic "container" for your agent.

For agents, the requirements.txt is as follows:

```
dynaconf==3.1.4
aio-pika
mythic-payloadtype-container==0.0.45
```

The `mythic-payloadtype-container` version of `0.0.45` corresponds to container version 9. Mythic now tracks the range of supported versions for all of the containers that connect up to it. This makes it easier to determine if you've updated mythic but don't have updated containers or visa versa.

### dynaconf

This is a super awesome addition that allows you to configure your PayloadType container with the `rabbitmq_config.json` file _OR_ via environment variables of the same names, but prefixed with `MYTHIC_`. So, if your rabbitmq instance isn't on the same host as your payload type container (or VM), you could either:

* Set the `host` key to the ip of your rabbitmq instance
* Set a `MYTHIC_HOST` environment variable to the ip of your rabbitmq instance

Either one of those will be pulled in (the environment configuration supersedes the json configuration).

## Building Payloads

The next set of changes come to how you build your agent. Nothing major changed, just some additional features and conventions changed since we're using a PyPi package now.

### builder.py

Your main agent defintion file in the `agentName/mythic/agent_functions/` (typically called builder.py) has some updates.

Replace your previous import:

```
from PayloadBuilder import *
```

with a new set of imports:

```python
from mythic_payloadtype_container.PayloadBuilder import *
from mythic_payloadtype_container.MythicCommandBase import *
```

There are two new things for your agent class that extends the functionality of your `PayloadType` - mythic\_encrypts and translation\_container.

#### mythic\_encrypts

This will be covered more in the `Translation Container` section, but your agent can now handle its own encryption/decryption if it wants. To signal this to Mythic, add `mythic_encrypts = False` in the same set of parameters where you declare the agent name, extension, and author.

#### translation\_container

If you want to have your own message format, do your own encryption, or generate your own keys, you need to have a "translation container". This will be covered more in depth in the [`Translation Container`](../../../customizing/payload-type-development/translation-containers.md) section, but if you have one for your payload type, declare it here with the name of the container. For example, if my translation container will be called "translator", then I'd add `translation_container = "translator"` to the same set of parameters where you declare the agent name, extension, and author.

There's a pre-existing Docker container you can use for this, `FROM itsafeaturemythic/python38_translator_container:0.0.3` or you can look at the corresponding GitHub repo ([https://github.com/MythicMeta/Mythic\_Docker\_Templates/tree/master/Docker\_Translation\_base\_files](https://github.com/MythicMeta/Mythic\_Docker\_Templates/tree/master/Docker\_Translation\_base\_files)) to create your own.

Translation containers, just like the other containers, have a simple `mythic_service.py` script to kick them off which consists of:

```python
#!/usr/bin/env python3
from mythic_translator_container import mythic_service
mythic_service.start_service_and_heartbeat(debug=False)
```

### building

#### C2 Parameters

Nothing fundamentally has changed with parsing C2 Profile Parameters for your agent, but there's some additional components. You still get an array (`self.c2info`) where each entry is related to one of the c2 profiles that the user is trying to add to your payload. For each of those entries you can still call the `get_c2profile()` function to get information about the profile and `get_parameters.dict().items()` to iterate over each key,value pair of parameter values. This is where the change happens:

**Crypto**

When doing crypto with Mythic, there used to be a lot of hard coded components, such as needing a parameter name to be exactly `AESPSK`. Now, that is no longer the case. C2 Profiles can declare any parameter to be a crypto related one simply by putting `crypto=True` in that C2 Profile's python definition file. As the payload creator though, what this means for you is that you don't just get a single base64 string back. The point here is to make crypto more modular and expansive, so you might get matching encryption/decryption keys, you might get a public key and a private key, you might get blank values to use for plaintext, etc. To suppor this, if a C2 profile parameter has `crypto=True`, then the `value` you get when iterating will be a `dictionary`. This dictionary has three components:

```
{ 
    "value": "string of whatever the user supplied when creating the agent",
    "enc_key": "base64 of the encryption key associated with that value",
    "dec_key": "base64 of the decryption key associated with that value"
}
```

What does this look like in practice? For the `http` profile, the crypto parameter gives the user a choice to select `aes256_hmac` or `none`. One of those two values will be the `value`, then there will either be `None` or the base64 of an aes256 key for the `enc_key` and `dec_key` components. You can then use these values however you need inside of your agent.

**Arrays / Dictionaries**

C2 Profiles can also provide a parameter type of `dictionary` which, when creating the agent, allows the operator to provide Key-Value pairs. This is useful for things like specifying Header values for HTTP (User-Agent, Host, etc). Instead of trying to deal with nested data structures for arrays vs single values in these dictionaries, the end result is an array of tuples. So, when creating your agent and looping through C2 parameter values, you might get an `array` of key-value pairs.

**Example**

Let's look at what this stuff all means with an example. For the `apfell` agent, stamping in C2 profile parameter values used to look like this:

```python
for c2 in self.c2info:
    profile = c2.get_c2profile()
    c2_code = open(
        self.agent_code_path
        / "c2_profiles"
        / "{}.js".format(profile["name"]),
        "r",
    ).read()
    for key, val in c2.get_parameters_dict().items():
        c2_code = c2_code.replace(key, val)
    all_c2_code += c2_code
base_code = base_code.replace("C2PROFILE_HERE", all_c2_code)
```

Now that we have a "dictionary" type for our `http` Profile's `headers` and we have the new crypto components, let's see what the section of code now looks like:

```python
for c2 in self.c2info:
    c2_code = ""
    try:
        profile = c2.get_c2profile()
        c2_code = open(
            self.agent_code_path
            / "c2_profiles"
            / "{}.js".format(profile["name"]),
            "r",
        ).read()
        for key, val in c2.get_parameters_dict().items():
            if isinstance(val, dict):
                c2_code = c2_code.replace(key, val["enc_key"] if val["enc_key"] is not None else "")
            elif not isinstance(val, str):
                c2_code = c2_code.replace(key, json.dumps(val))
            else:
                c2_code = c2_code.replace(key, val)
    except Exception as p:
        build_msg += str(p)
    all_c2_code += c2_code
base_code = base_code.replace("C2PROFILE_HERE", all_c2_code)
```

Here we're checking if the value is an instance of `dict` which means we're looking at crypto information, we check if the value is a `str` which is the normal other data, and if it's neither of those, then it's the array of dictionaires for our header values. You can of course also check if the `key` matches the name of the C2 Profile Parameter values (`AESPSK` and `headers`) and do your check that way too.

#### Build status

The last piece that's updated about building is that you can now be more explicit about what's going on. Rather than only being able to report back success/failure and a single message (which can get really messy with debug output), you can now set the following in your `BuildResponse` object:

* build\_message - this is a simple standard message you display to the user when things build correctly (either `myBuildResp.build_message = "congrats, new agent created"` or `myBuildResp.set_build_message("congrats, new agent created")`.
* build\_stderr - this is error information if something goes wrong (either `myBuildResp.build_stderr = "compile error here"` or `myBuildResp.set_build_stderr("compile error here")`.
* build\_stdout - this is helpful stdout information that you might not want to present to the user on success, but would be helpful to look at later on (either `myBuildResp.build_stdout = "additional debugging info here"` or `myBuildResp.set_build_stdout("additional debugging info here")`. )

{% hint style="warning" %}
The old style of payloads used `message` instead of `build_message`, so you will have to change that one.
{% endhint %}

## Commands

There are a lot of small updates with commands that give us a lot of new features, so let's dive into those. These sections are talking about all of the individual command files you have in `agentName/mythic/agent_functions/commandName.py`. This section will use Apfell's `shell` command as an example.

* 2.1 version - [https://github.com/its-a-feature/Mythic/blob/master/Payload\_Types/apfell/mythic/agent\_functions/shell.py](https://github.com/its-a-feature/Mythic/blob/master/Payload\_Types/apfell/mythic/agent\_functions/shell.py)
* 2.2 version - [https://github.com/MythicAgents/apfell/blob/master/Payload\_Type/apfell/mythic/agent\_functions/shell.py](https://github.com/MythicAgents/apfell/blob/master/Payload\_Type/apfell/mythic/agent\_functions/shell.py)

### PyPi imports

Just like with the payload building, we need to change our imports at the top. There are two main changes here. First, the `from CommandBase import *` needs to change to `from mythic_payloadtype_container.MythicCommandBase import *`. Secondly, if you were importing various files for RPC, such as `from MythicResponseRPC import *`, those have all been collapsed into a single file now, so you'd need to import `from mythic_payloadtype_container.MythicRPC import *`.

### CommandParameter

Not much changed here, but there are a few things to call out:

* CommandParameter's now have a `ui_position` attribute you can set to order your arguments in a specific way
*   The `name` attribute does _NOT_ have to match the value of the args key. The `name` attribute is what's presented to the user for the short name of the parameter, the `description` is what's displayed when the user hovers over that short name, and the dictionary key that's associated with the `CommandParameter` object overall is what's used when sending information down to the agent. Let's take an example:

    ```
    self.args = {
      "file": CommandParameter(
          name="Select a File", type=ParameterType.File, description="file to upload"
      ),
    }
    ```

    In this case, the user would see in their popup modal `Select a File` with a file selection button. If they hovered their mouse over `Select a File`, they'd see the description, `file to upload`, and when the data goes down to the agent, the agent will get `{"file": "uuid here"}`. When interacting with these kinds of mis-matched names from the `create_tasking` function, you want to reference the dictionary key value, not the `name` value (i.e. `task.args.get_arg("file")`).
* All the different kinds of Parameter types can be found on the Public Github [https://github.com/MythicMeta/Mythic\_PayloadType\_Container](https://github.com/MythicMeta/Mythic\_PayloadType\_Container).

When using a parameter type of `ChooseOne` or `ChooseMultiple`, choices is an array of choices for the user. If your command needs you to pick from the set of commands (rather than a static set of values), then there are a few other components that come into play. If you want the user to be able to select any command for this payload type, then set `choices_are_all_commands` to True. Alternatively, you could specify that you only want the user to choose from commands that are already loaded into the callback, then you'd set `choices_are_loaded_commands` to True. As a modifier to either of these, you can set `choice_filter_by_command_attributes` to filter down the options presented to the user even more based on the parameters of the Command's attributes parameter. This would allow you to limit the user's list down to commands that are loaded into the current callback that support MacOS for example. An example of this would be:

```python
CommandParameter(name="test name", 
                 type=ParameterType.ChooseMultiple, 
                 description="so many choices!", 
                 choices_are_all_commands=True,
                 choice_filter_by_command_attributes={"supported_os": [SupportedOS.MacOS]})
```

### CommandOPSEC

Just like how there's a `TaskArgments` class and a `CommandBase` class, there's a `CommandOPSEC` class now that you can implement for your commands. This will expand over time, but for now there are a few things you can do:

* There are currently three tracked attributes about a command - `injection_method`, `process_creation`, and `authentication` which all are free-form text fields that you can set to help describe what all your command might be doing on host that's an OPSEC consideration
* implement the `opsec_pre` and `opsec_post` functions. This is more detailed, so let's take the next few sections to walk through an example.

Your instance of CommandOpsec, is then tied to your command in the same way your instance of TaskArgument is - simply add `opsec_class = ShellOPSEC` (but the name of your Subclass) to the same area where you have `cmd`, `needs_admin`, and `description` in your command class.

#### opsec\_pre

This function `async def opsec_pre(self, task: MythicTask)` if implemented, is called _before_ a task's `create_tasking` function. The point of this function is to do some operational security pre-flight tests before passing execution on to your `create_tasking` function.

Let's take an example - you're operating in an environment and are about to run a command that does spawn and inject. Before you do that command, you want to query what you know so far about the environment to see if there are any EDR products running that might alert on that activity. So, before even passing execution to the `create_tasking` function, in the `opsec_pre` function, you can use RPC calls back to Mythic to query information. In our example, let's query the `Process` table to see if we have any process data for the host where our task is running. We can do this with a simple RPC call:

```
processes = await MythicRPC().execute("search_database", task_id=task.id, table="process", host=task.callback.host)
```

`processes` now has a response object back from Mythic that holds some information:

* processes.status - this is a MythicStatus value that indicates `Success` or `Error` for the RPC call overall
* processes.error\_message - if the status is `MythicStatus.Error`, then this is populated with the error message
* processes.response - this is the actual response we got back from Mythic. This will vary wildly with the function call that you did.

Now that we have the basics, let's expand this example out to say that if we don't have any process data, we should block the function execution until we do have process data. If we do have processes, we should do another query to see if any of the processes match something dangerous, like `Microsoft Defender`:

```python
async def opsec_pre(self, task: MythicTask):

    processes = await MythicRPC().execute("search_database", task_id=task.id, table="process",
                                          host=task.callback.host)
    if processes.status == MythicStatus.Success:
        if len(processes.response) == 0:
            task.opsec_pre_blocked = True
            task.opsec_pre_message = f"This spawns {self.process_creation} and there is no process data on the host yet."
            task.opsec_pre_message += "\nRun \"list_apps\" first to check for dangerous processes"
            task.opsec_pre_bypass_role = "operator"
            return
        else:
            processes = await MythicRPC().execute("search_database", task_id=task.id, table="process",
                                                  name="Microsoft Defender", host=task.callback.host)
            if len(processes.response) > 0:
                task.opsec_pre_blocked = True
                task.opsec_pre_message = f"Microsoft Defender spotted on the host in running processes. Don't spawn commands this way"
    else:
        task.opsec_pre_blocked = True
        task.opsec_pre_message = f"Failed to query processes from Mythic:\n{processes}"
```

You'll notice that we are setting a few fields on the task depending on what we see:

* task.opsec\_pre\_blocked = True - we set this if we want to stop execution here and report something back to the user
* task.opsec\_pre\_message - this is the message we want to send to the user

All of the RPC calls that are available during create\_tasking are available here as well.

* task.opsec\_pre\_bypass\_role - we can set this to "operator" or "lead" based on who should be allowed to bypass this opsec issue.

If a message is blocked, the task status in the UI will indicate it's blocked. When you click on the task status there will be two additional menu options - submit a bypass request and view the message.

#### opsec\_post

This is very similar to `opsec_pre`, except it happens _after_ the `create_tasking` call. This is useful for when you're generating artifacts as part of your tasking (such as generating new DLLs) and want to make sure they're properly sanitized or obfuscated before allowing an agent to pick it up. All of the same components apply, it's just `opsec_post_*`.

### Command Attributes

This will expand over time, but currently we're adding in an `attributes` component to Commands that describes non-opsec related attributes. Currently, this only includes two things: if a command is spawn\_and\_injectable and what kind of operating systems the command supports.

This looks like the following:

```python
attributes = CommandAttributes(
        spawn_and_injectable=True,
        supported_os=[SupportedOS.MacOS]
    )
```

The `supported_os` attribute lists the same supported OS types as the Payload Type. Most of the time, these two will match up; however, if your agent can compile to multiple different operating systems, this is one way to make it so that during payload creation, the user can only see the commands that are associated with the kind of payload they're trying to make. The spawn\_and\_injectable variable helps provide some quality of life to operators in case they try to inject a command like "exit" or "cd" into a remote process, which doesn't really make sense.

### supported\_ui\_features

There are a bunch of features within the Mythic UI that look for specific agent functions to call. Historically, this was indicated with a series of boolean values - `is_exit`, `is_process_list`, etc. That was fine initially, but doesn't make it easy to expand. So, all of these are now grouped up into a new attribute called `ui_features` in a way that's more explicit about _where_ a function will be referenced. These are now mapped as follows:

```python
is_exit = True -> supported_ui_features = ["callback_table:exit"]
is_file_browse = True -> supported_ui_features = ["file_browser:list"]
is_process_list = True -> supported_ui_features = ["process_browser:list"]
is_download_file = True -> supported_ui_features = ["file_browser:download"]
is_remove_file = True -> supported_ui_features = ["file_browser:remove"]
is_upload_file = True -> supported_ui_features = ["file_browser:upload"]
```

This also more easily allows us to expand this going forward for future UI elements and allows us to eventually leverage a single command in multiple areas more explicitly. If none of those `is_*` attributes are `True`, then you don't even need to supply the `supported_ui_features` attribute.

This now allows us to go from:

```python
is_exit = False
is_file_browse = True
is_process_list = False
is_download_file = False
is_remove_file = False
is_upload_file = False
```

to

```python
supported_ui_features = ["file_browser:list"]
```

which is way more descriptive about what and where the command is used.

### create\_tasking

The `create_tasking` function got a few updates as well. Firstly, all of the calls for RPC functionality has changed. It used to be the case where you had to know which RPC function you want to execute, you had to know which RPC file had it, and you had to know all of the parameters for it. This gets complicated fast, and there isn't always a clear location for a function. For example, where would a `search_database` function go?

To help with this, there is now a single RPC file, `from mythic_payloadtype_container.MythicRPC import *` that has all of the functionality within it. Every function call will be of the form:

```python
response = await MythicRPC().execute("function name", argument1=val, argument2=val, ...etc)
```

You can programmatically get all of the available functions, their prototypes, and their docstrings by running the `get_functions` function. More information and all of the current 2.2.2 functions can be found on the [MythicRPC](mythicrpc.md) page:

```python
from mythic_payloadtype_container.MythicRPC import *
import sys
resp = await MythicRPC().get_functions()
print(resp.response)
sys.stdout.flush()
```

This will go through to query the Mythic server for all of the functions that are exposed via RPC and give detailed information for them.

#### stdout/stderr

For each task, you can now track the stdout and stderr for later reference. Simply set it via:

```
task.stdout = "something here"
task.stderr = "my error messages here"
```

Then, you can see this via the UI when you select a task's status. This is helpful when you're doing commands that might result in additional compilation or analysis (such as load creating new modules).

#### display\_params

For the agent and for Mythic, using structured arguments and output is extremely beneficial. However, from the operator standpoint, this is hard to display, hard to quick glance at, and quickly bloats the screen. To help with this, tasks can set their own `display_params` by simply doing:

```
task.display_params = task.args.get_arg("my arg") + " some other words"
```

Then, once your `create_tasking` function returns, the commandline information that the operator sees is updated to this new custom value. If you select the status for the task though, you can select to "view all parameters" - this will create a new popup with information for three different stages of parameters:

```
original params - this is what's sent to the Mythic server when issuing a task, whatever form that happens to be
display params - this is the same as original params by default and changes if you update it via task.display_params
final params - this is what actually gets sent down to the agent (this is hepful to see in case you do some manipluation or setting of parameters via the argument parsing or create_tasking)
```

### process\_response

The `process_response` function is now finally callable within the Command files. The point of this function is to have a custom, programmatic execution based on the output of a command. When reporting data back from your agent, in your `post_response` array (the same place you'd set `user_output`), you can specify `process_response` with whatever data you want. This is then wrapped up with the task information into an `AgentResponse` object with two attributes:

* task - this is just the same task information you get in `create_tasking`
* response - this is the data you put in the `process_response` key.

An example of using this to update a callback's sleep information is:

```
async def process_response(self, response: AgentResponse):
    resp = await MythicRPC().execute("update_callback", sleep_info=response.response)
```

This function also has access to all of the same RPC functionality that the `create_tasking` has. Here we're simply updating the callback's `sleep_info` with the result of the response. This allows us to programmatically update it based on if the agent was successful or not, without requiring Mythic to expose a custom response attribute for this field.

### Dynamic Parameter Values

[Dynamic Parameter Values](../../../customizing/payload-type-development/dynamic-parameter-values.md)

### Sub-Tasking / Task Callbacks

[Sub-tasking](../../../customizing/payload-type-development/sub-tasking-task-callbacks.md)

### Tags

[Tags](../../../operational-pieces/tags.md)

### Script\_Only Commands

To go along with the sub-tasking and task callback functionality, you might run into scenarios where you want to provide some sort of dynamic check or execution within the Command file, but don't it to actually be a task that gets sent down to the agent. You can now specify `script_only=True` in your Command file and this will be exactly the case.

Commands that are marked as `script_only=True` will NOT appear when you go to build a payload because these commands are transparent to your agent. They WILL appear in the type hints when you start typing commands though and they appear in the list of full available commands when you view metadata about a callback.

You might be wondering why this is useful? Consider the following: `psexec` is a command you want to implement in your agent, but you implement this functionality manually rather than simply running the Microsoft psexec binary. You can create a `script_only` `psexec` command that when a user types it, will spin of further sub-tasks to check that the computer is reachable, that the port is open, that you have access, copies over the file, creates the service, then cleans it all up. The `psexec` command in that case is kind of like a conductor that controls all the other tasks, switches based on success/error for each sub-task, and ultimately accomplishes the task dynamically without needing the command itself to be a compiled specific task in your agent.
