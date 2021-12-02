# Payload Type Info

Payload Type information must be set and pulled from python within the associated `/Mythic/Payload_Types/[agent name]/mythic/agent_functions` folder. The filename can be anything, but the contents should look like:

```python
from mythic_payloadtype_container.MythicCommandBase import *
from mythic_payloadtype_container.PayloadBuilder import *


class Atlas(PayloadType):  # class name can be anything

    name = "atlas"  # name of your agent and name of your folder must match
    file_extension = "exe"
    author = "@Airzero24"
    supported_os = [
        SupportedOS.Windows
    ]
    wrapper = False
    mythic_encrypts = True
    wrapped_payloads = ["service_wrapper"]
    note = """This payload uses C# to target Windows hosts with the .NET framework installed. For more information and a more detailed README, check out: https://github.com/airzero24/Atlas"""
    supports_dynamic_loading = False
    build_parameters = [
        BuildParameter(
            name="version",
            parameter_type=BuildParameterType.ChooseOne,
            description="Choose a target .NET Framework",
            choices=["4.0", "3.5"],
        ),
        BuildParameter(
            name="arch",
            parameter_type=BuildParameterType.ChooseOne,
            choices=["x64", "x86"],
            default_value="x64",
            description="Target architecture",
        ),
        BuildParameter(
            name="chunk_size",
            parameter_type=BuildParameterType.String,
            default_value="512000",
            description="Provide a chunk size for large files",
            required=True,
        ),
        BuildParameter(
            name="cert_pinning",
            parameter_type=BuildParameterType.Boolean,
            default_value=False,
            required=False,
            description="Require Certificate Pinning",
        ),
        ...
    ]
    c2_profiles = ["http"]
    translation_container = None
    
    async def build(self) -> BuildResponse:
        # this function gets called to create an instance of your payload
        resp = BuildResponse(status=BuildStatus.Error)
        current_config_name = "{}.cs".format(str(uuid.uuid4()))
        current_config_path = "{}/{}".format(self.agent_code_path, current_config_name)
        # create the payload
        ...
        return resp

```

There are a couple key pieces of information here:

* Line 1-2 imports all of basic classes needed for creating an agent
* line 5 defines the new class (our agent). This can be called whatever you want, but the important piece is that it extends the `PayloadType` class as shown with the `()`.
* Lines 6-48 defines the parameters for the payload type that you'd see throughout the UI.
  * the name is the name of the payload type
  * supported\_os is an array of supported OS versions
  * supports\_dynamic\_loading indicates if the agent allows you to select only a subset of commands when creating an agent or not
  * build\_parameters is a dictionary describing all build parameters when creating an agent
    * the "key" here and the "name" in the BuildParameter() class must match.
  * c2\_profiles is an array of c2 profile names that the agent supports
* Line 49 defines the name of a "translation container" which we will talk about in another section, but this allows you to support your own, non-mythic message format, custom crypto, etc.
* The last piece is the function that's called to **build** the agent based on all of the information the user provides from the web UI.

The `PayloadType` base class is in the `PayloadBuilder.py` file. This is an abstract class, so your instance needs to provide values for all these fields.

## Build Parameters

Build parameters define the components shown to the user when creating a payload.

![](<../../.gitbook/assets/Screen Shot 2021-12-02 at 11.38.16 AM.png>)

The `BuildParameter` class has a couple of pieces of information that you can use to customize and validate the parameters supplied to your build:

```python
class BuildParameter:

    def __init__(self,
                 name: str,
                 parameter_type: BuildParameterType = None,
                 description: str = None,
                 required: bool = None,
                 verifier_regex: str = None,
                 default_value: str = None,
                 choices: [str] = None,
                 value: any = None,
                 verifier_func: callable = None
                 ):
        self.name = name
        self.verifier_func = verifier_func
        self.parameter_type = parameter_type if parameter_type is not None else ParameterType.String
        self.description = description if description is not None else ""
        self.required = required if required is not None else True
        self.verifier_regex = verifier_regex if verifier_regex is not None else ""
        self.default_value = default_value
        if value is None:
            self.value = default_value
        else:
            self.value = value
        self.choices = choices
```

* name is the name of the parameter, if you don't provide a longer description, then this is what's presented to the user when building your payload
* parameter\_type describes what is presented to the user - a String or a ChooseOne dropdown list
* required indicates if there must be a value supplied. If no value is supplied by the user and no default value supplied here, then an exception is thrown before execution gets to the `build` function.
* verifier\_regex is a regex the web UI can use to provide some information to the user about if they're providing a valid value or not
* default\_value is the default value used for building if the user doesn't supply anything
* choices is where you can supply an array of options for the user to pick from if the parameter\_type is ChooseOne
* value is the component you access when building your payload - this is the final value (either the default value or the value the user supplied)
* verifier\_func is a function you can provide for additional checks on the value the user supplies to make sure it's what you want. This function should either return nothing or raise an exception if something isn't right

As a recap, where does this come into play? In the first section, we showed a section like:

```python
build_parameters = [
    BuildParameter(
        name="version",
        parameter_type=BuildParameterType.ChooseOne,
        description="Choose a target .NET Framework",
        choices=["4.0", "3.5"],
    ),
    BuildParameter(
        name="arch",
        parameter_type=BuildParameterType.ChooseOne,
        choices=["x64", "x86"],
        default_value="x64",
        description="Target architecture",
    ),
    BuildParameter(
        name="chunk_size",
        parameter_type=BuildParameterType.String,
        default_value="512000",
        description="Provide a chunk size for large files",
        required=True,
    ),
    BuildParameter(
        name="cert_pinning",
        parameter_type=BuildParameterType.Boolean,
        default_value=False,
        required=False,
        description="Require Certificate Pinning",
    ),
    ...
]
```

## Building

You have to implement the `build` function and return an instance of the `BuildResponse` class. This response has three fields:

* status - an instance of BuildStatus (Success or Error)
  * Specifically, `BuildStatus.Success` or `BuildStatus.Error`
* payload - the raw bytes of the finished payload (if you failed to build, set this to `None` or empty bytes like `b''` in Python.
* build\_message - any stdout data you want the user to see
* build\_stderr - any stderr data you want the user to see
* build\_stdout - any stdout data you want the user to see

The most basic version of the build function would be:

```python
async def build(self) -> BuildResponse:
        # this function gets called to create an instance of your payload
        return BuildResponse(status=BuildStatus.Success)
```

Once the `build` function is called, all of your `BuildParameters` will already be verified (all parameters marked as `required` will have a `value` of some form (user supplied or default\_value) and all of the verifier functions will be called if they exist). This allows you to _know_ that by the time your `build` function is called that all of your parameters are valid.

Your build function gets a few pieces of information to help you build the agent (other than the build parameters):

From within your build function, you'll have access to the following pieces of information:

* `self.uuid` - the UUID associated with your payload
  * This is how your payload identifies itself to Mythic before getting a new Staging and final Callback UUID
* `self.commands` - a wrapper class around the names of all the commands the user selected.
  * Access this list via `self.commands.get_commands()`

```python
for cmd in self.commands.get_commands():
    command_code += open(self.agent_code_path / "{}.js".format(cmd), 'r').read() + "\n"
```

* `self.agent_code_path` - a `pathlib.Path` object pointing to the path of the `agent_code` directory that holds all the code for your payload. To be more explicit, this is the path `/Mythic/agent_code` in the normal docker containers.
  * To access "test.js" in that "agent\_code" folder, simply do:\
    `f = open(self.agent_code_path / "test.js", 'r')`.
  * With `pathlib.Path` objects, the `/` operator allows you to concatenate paths in an OS agnostic manner. This is the recommended way to access files so that your code can work anywhere.
* `self.get_parameter("parameter name here")`
  * The build parameters that are validated from the user. If you have a build\_parameter with a name of "version", you can access the user supplied or default value with `self.get_parameter("version")`
* `self.selected_os` - This is the OS that was selected on the first step of creating a payload
* `self.c2info` - this holds a list of dictionaries of the c2 parameters and c2 class information supplied by the user. This is a list because the user can select multiple c2 profiles (maybe they want HTTP and SMB in the payload for example). For each element in self.c2info, you can access the information about the c2 profile with `get_c2profile()` and access to the parameters via `get_parameters_dict()`. Both of these return a dictionary of key-value pairs.
*
  * the dictionary returned by `self.c2info[0].get_c2profile()` contains the following:
    * `name` - name of the c2 profile
    * `description` - description of the profile
    * `is_p2p` - boolean of if the profile is marked as a p2p profile or not
    * `is_server_routed` - boolean of if the profile is server routed or not
  * the dictionary returned by `self.c2info[0].get_parameters_dict()`contains the following:
    * `key` - value
    * where each `key` is the `key` value defined for the c2 profile's parameters and `value` is what the user supplied. You might be wondering where to get these keys? Well, it's not too crazy and you can view them right in the UI - [Name Fields](../../scripting/scripting-payloads.md#how-do-you-find-the-name-fields).
    * If the C2 parameter has a value of `crypto_type=True`, then the "value" here will be a bit more than just a string that the user supplied. Instead, it'll be a dictionary with three pieces of information: `value` - the value that the user supplied, `enc_key` - a base64 string (or None) of the encryption key to be used, `dec_key` - a base64 string (or None) of the decryption key to be used. This gives you more flexibility in automatically generating encryption/decryption keys and supporting crypto types/schemas that Mythic isn't aware of. In the HTTP profile, the key `AESPSK` has this type set to True, so you'd expect that dictionary.
    * If the C2 parameter has a type of "Dictionary", then things are a little different. The dictionary is represented by an array of entries, which allows you to have multiple values mapped to the single dictionary value without worrying about complex UI elements to support dynamic key/value generation. What does this mean for you, the dev though?
      * Let's take the "headers" parameter in the `http` profile for example. This allows you to set header values for your `http` traffic such as User-Agent, Host, and more. When you get this value on the agent side, you get an array of values that look like the following:\
        `[ {"name": "User-Agent"}, "key": "User-Agent", "value": "the user agent the user supplied", "custom": false}, {"name": "*", "key": "MyCustomHeader", "value": "my custom value", "custom": true} ]`. So, what are we looking at? When you define a "Dictionary" type C2 Profile parameter you can specify default values, the maximum number of times a key can appear, and even allow user "write-ins" that you didn't explicitly specify as options. That's what we're seeing here: the "User-Agent" key wasn't custom created, but a "myCustomHeader" key that was custom. The "name" key is what's presented to the user by default - a value of "\*" means that the user can write anything in for that "key".
  * One way to leverage this could be:

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
```

Finally, when building a payload, it can often be helpful to have both stdout and stderr information captured, especially if you're compiling code. Because of this, you can set the `build_essage` ,`build_stderr` , and `build_stdout` fields of the `BuildResponse` to have this data. For example:

```python
async def build(self) -> BuildResponse:
    # this function gets called to create an instance of your payload
    resp = BuildResponse(status=BuildStatus.Success)
    # create the payload
    build_msg = ""
    try:
        command_code = ""
        for cmd in self.commands.get_commands():
            command_code += (
                open(self.agent_code_path / "{}.js".format(cmd), "r").read() + "\n"
            )
        base_code = open(
            self.agent_code_path / "base" / "apfell-jxa.js", "r"
        ).read()
        base_code = base_code.replace("UUID_HERE", self.uuid)
        base_code = base_code.replace("COMMANDS_HERE", command_code)
        all_c2_code = ""
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
                    else:
                        c2_code = c2_code.replace(key, val)
            except Exception as p:
                build_msg += str(p)
            all_c2_code += c2_code
        base_code = base_code.replace("C2PROFILE_HERE", all_c2_code)
        resp.payload = base_code.encode()
        resp.build_message = "Successfully built!\n"
        resp.build_stderr = build_msg
    except Exception as e:
        resp.set_status(BuildStatus.Error)
        resp.build_stderr = "Error building payload: " + str(e)
    return resp
```

Depending on the status of your build (success or error), either the message or build\_stderr values will be presented to the user via the UI notifications. However, at any time you can go back to the Created Payloads page and view the build message, build errors, and build stdout for any payload.

{% hint style="info" %}
When building your payload, if you have to modify files on disk, then it's helpful to do this in a "copy" of the files. You can make a temporary copy of your code and operate there with the following sample:

```python
agent_build_path = tempfile.TemporaryDirectory(suffix=self.uuid)
# shutil to copy payload files over
copy_tree(self.agent_code_path, agent_build_path.name)
# now agent_build_path.name maps to the root folder for your agent code
```
{% endhint %}

## Execution flow

So, what's the actual, end-to-end execution flow that goes on? A diagram can be found here: [#what-happens-for-building-payloads](../../message-flow.md#what-happens-for-building-payloads "mention").

1. PayloadType container is started, it connects to Mythic and sends over its data (by parsing all these python files)
2. An operator wants to create a payload from it, so they go to "Create Components" -> "Create Payload"
3. The operator selects an OS type that the agent supports (ex. Linux, macOS, Windows)
4. The operator selects all c2 profiles they want included
   1. and for each c2 selected, provides any c2 required parameters
5. The operator selects this payload type
6. The operator fills out/selects all of the payload type's build parameters
7. The operator selects all commands they want included in the payload
8. Mythic takes all of this information and sends it to the payload type container

The mythic\_service in the Payload Type container does the following:

```python
c2info_list = []
for c2 in message_json['c2_profile_parameters']:
    params = c2.pop('parameters', None)
    c2info_list.append(C2ProfileParameters(parameters=params, c2profile=c2))
commands = CommandList(message_json['commands'])
for cls in PayloadType.__subclasses__():
        agent_builder = cls(uuid=message_json['uuid'],
                            agent_code_path=Path(container_files_path),
                            c2info=c2info_list,
                            commands=commands,
                            wrapped_payload=message_json['wrapped_payload'])
try:
    await agent_builder.set_and_validate_build_parameters(message_json['build_parameters'])
    build_resp = await agent_builder.build()
```

Specifically, it takes the information that the user selected for the c2 profiles and gets them organized into a list (lines 1-4). It makes a list of all of the selected commands (line 5). It looks for a class that subclasses the PayloadType class and instantiates it (lines 6-11). It takes the user supplied parameters, sets the associated build parameter values, and **verifies** that all build parameters marked as "required" have values (line 13). Lastly, it calls the PayloadType's `build` function.

After this section, the container sends the `BuildResponse` message back to the Mythic server.
