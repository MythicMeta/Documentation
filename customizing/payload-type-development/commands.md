# Commands

## What do Commands track?

Command information is tracked in your Payload Type's container within the `mythic/agent_functions` folder. Each command has its own Python file with a few classes in it. Specifically, things are broken out into three classes:

* CommandBase
* TaskArguments
* CommandOPSEC

**CommandBase** defines the metadata about the command as well as any pre-processing functionality that takes place before the final command is ready for the agent to process. This class includes the `create_tasking` ([Create\_Tasking](create\_tasking.md)) and `process_response` ([Process Response](process-response.md)) functions.&#x20;

****[**TaskArguments**](commands.md#taskarguments) does two things:

1. defines the parameters that the command needs
2. verifies / parses out the user supplied arguments into their proper components
   * this includes taking user supplied free-form input (like arguments to a sleep command - `10 4`) and parsing it into well-defined JSON that's easier for the agent to handle (like `{"interval": 10, "jitter": 4}`)
   * This also includes verifying all the necessary pieces are present. Maybe your command requires a source and destination, but the user only supplied a source. This is where that would be determined and error out for the user. This prevents you from requiring your agent to do that sort of parsing in the agent.

**CommandOPSEC (**[**OPSEC Checking**](opsec-checking.md)**) **defines how your command behaves from an OPSEC perspective and allows scripting points (`opsec_pre` and `opsec_post`) to perform dynamic checks and blocks for users.

## CommandBase

```python
from mythic_payloadtype_container.PayloadBuilder import *
from mythic_payloadtype_container.MythicCommandBase import *

class ScreenshotCommand(CommandBase):
    cmd = "screenshot"
    needs_admin = False
    help_cmd = "screenshot"
    description = "Use the built-in CGDisplay API calls to capture the display and send it back over the C2 channel. No need to specify any parameters as the current time will be used as the file name"
    version = 1
    author = ""
    attackmapping = ["T1113"]
    argument_class = ScreenshotArguments
    browser_script = BrowserScript(script_name="screenshot", author="@its_a_feature_")
    attributes = CommandAttributes(
        spawn_and_injectable=True,
        supported_os=[SupportedOS.MacOS],
        builtin=False,
        load_only=False,
        suggested_command=False,
        
    )
    script_only = False
    
    async def create_tasking(self, task: MythicTask) -> MythicTask:
        task.args.command_line += str(datetime.datetime.utcnow())
        return task
        
    async def process_response(self, response: AgentResponse):                                                    add=response.response, remove=[])
        pass
```

Creating your own command requires extending this CommandBase class (i.e. `class ScreenshotCommand(CommandBase)` and providing values for all of the above components.

* `cmd` - this is the command name. The name of the class doesn't matter, it's this value that's used to look up the right command at tasking time
* `needs_admin` - this is a boolean indicator for if this command requires admin permissions
* `help_cmd` - this is the help information presented to the user if they type `help [command name]` from the main active callbacks page
* `description` - this is the description of the command. This is also presented to the user when they type help.
* `suported_ui_features` - This is an array of values that indicates where this command might be used within the UI. For example, from the active callbacks page, you see a table of all the callbacks. As part of this, there's a dropdown you can use to automatically issue an `exit` task to the callback. How does Mythic know which command to actually send? It's this array that dictates that. The following are used by the callback table, file browser, and process listing, but you're able to add in any that you want and leverage them via browser scripts for additional tasking:
  * supported\_ui\_features = \["callback\_table:exit"]
  * supported\_ui\_features = \["file\_browser:list"]&#x20;
  * supported\_ui\_features = \["process\_browser:list"]
  * supported\_ui\_features = \["file\_browser:download"]&#x20;
  * supported\_ui\_features = \["file\_browser:remove"]&#x20;
  * supported\_ui\_features = \["file\_browser:upload"]
* `version` - this is the version of the command you're creating/editing. This allows a helpful way to make sure your commands are up to date and tracking changes
* `argument_class` - this correlates this command to a specific `TaskArguments` class for processing/validating arguments
* `attackmapping` - this is a list of strings to indicate MITRE ATT\&CK mappings. These are in "T1113" format.
* `agent_code_path` is automatically populated for you like in building the payload. This allows you to access code files from within commands in case you need to access files, functions, or create new pieces of payloads. This is really useful for a `load` command so that you can find and read the functions you're wanting to load in.
* You can optionally add in the `attributes` variable. This is a new class called `CommandAttributes` where you can set whether or not your command supports being injected into a new process (some commands like `cd` or `exit` don't make sense for example). You can also provide a list of supported operating systems. This is helpful when you have a payload type that might compile into multiple different operating system types, but not all the commands work for all the possible operating systems. Instead of having to write "not implemented" or "not supported" function stubs, this will allow you to completely filter this capability out of the UI so users don't even see it as an option.&#x20;
  * Available options are:
    * `supported_os` an array of SupportedOS fields (ex: `[SupportedOS.MacOS]`)
    * `spawn_and_injectable` is a boolean to indicate if the command can be injected into another process
    * `builtin` is a boolean to indicate if the command should be always included in the build process and can't be unselected
    * `load_only` is a boolean to indicate if the command can't be built in at the time of payload creation, but can be loaded in later
    * `suggested_command` is a boolean to indicate if the command should be pre-selected for users when building a payload
    * `filter_by_build_parameter` is a dictionary of `parameter_name:value` for what's required of the agent's build parameters. This is useful for when some commands are only available depending on certain values when building your agent (such as agent version).&#x20;
  * This ties into the CommandParameter fields `choice_filter_by_command_attributes`, `choices_are_all_commands`, and `choices_are_loaded_commands`.
* The `create_tasking` function is very broad and covered in [Create\_Tasking](create\_tasking.md#create\_tasking)
* The `process_response` is similar, but allows you to specify that data shouldn't automatically be processed by Mythic when an agent checks in, but instead should be passed to this function for further processes and to use Mythic's RPC functionality to register the results into the system. The data passed here comes from the `post_response` message ([Process Response](process-response.md)).
* The `script_only` flag indicates if this Command will be use strictly for things like issuing [subtasking](sub-tasking-task-callbacks.md), but will NOT be compiled into the agent. The nice thing here is that you can now generate commands that don't need to be compiled into the agent for you to execute. These tasks never enter the "submitted" stage for an agent to pick up - instead they simply go into the [create\_tasking](create\_tasking.md)  scenario (complete with subtasks and full RPC functionality) and then go into a completed state.&#x20;

## TaskArguments

The TaskArguments class defines the arguments for a command and defines how to parse the user supplied string so that we can verify that all required arguments are supplied.

Task arguments always take in a string `command_line` from the UI and either keep it as-is or parse it into `CommandParameter` classes. This is **ALWAYS** a string for `command_line`, so even if you use the popup UI and supply your parameters that way, that JSON will get turned into a JSON string that's then passed here. Below is an example of passing arguments to the `ls` command for the `apfell` payload.&#x20;

In `self.args` we define a dictionary of our arguments and what they should be along with default values if none were provided.&#x20;

In `parse_arguments` we parse the user supplied `self.command_line` into the appropriate arguments. If the user did everything through the UI popup menu, then it should be as easy as `self.load_args_from_json_string(self.command_line)`, the harder part comes when you allow the user to type arguments free-form and then must parse them out into the appropriate pieces.

```python
class LsArguments(TaskArguments):
    def __init__(self, command_line):
        super().__init__(command_line)
        self.args = {
            "path": CommandParameter(name="path", type=ParameterType.String, default_value=".")
        }

    async def parse_arguments(self):
        if len(self.command_line) > 0:
            if self.command_line[0] == '{':
                self.load_args_from_json_string(self.command_line)
            else:
                self.add_arg("path", self.command_line)
```

{% hint style="info" %}
You might look at the above code snippet and see things like "path" twice. That's not a hard requirement. The "key" in the `self.args` dictionary is how you look up the command parameters when you're creating your tasking. the "name" in `name="path"` is what's displayed to the user as they're filling out these parameters. So, you could have a key of "path" but a name of "Remote Path".
{% endhint %}

The main purpose of the TaskArguments class is to manage arguments for a command. It handles parsing the `command_line` string into `CommandParameters`, defining the `CommandParameters`, and providing an easy interface into updating/accessing/adding/removing arguments as needed.

The class **must** implement the `parse_arguments` method and define the `args` dictionary. This `parse_arguments` method is the one that allows users to supply "short hand" tasking and still parse out the parameters into the required JSON structured input.&#x20;

When syncing the command with the UI, Mythic goes through each class that extends the CommandBase, looks at the associated `argument_class`, and parses that class's `args` dictionary of `CommandParameters` to create the pop-up in the UI. While the TaskArgument's `parse_arguments` method simply parses the user supplied input into JSON, it's the CommandParameter's class that actually verifies that every required parameter has a value, that all the values are appropriate, and that default values are supplied if necessary.

### CommandParameters

CommandParameters, similar to BuildParameters, provide information for the user via the UI and validates that the values are all supplied and appropriate.

```python
class CommandParameter:
    name: str  # name of the parameter
    type: ParameterType  # type of the parameter
    description: str  # description of the parameter
    choices: [any]  # choices if the type is ParameterType.ChooseOne or ParameterType.ChooseMultiple
    required: bool  # if the parameter is required or not
    validation_func: callable  # a function to validate if the supplied value is good
    value: any  # the user supplied value
    default_value: any  # a default value to use if the user didn't supply anything

    def __init__(self, name: str,
                 type: ParameterType,
                 description: str = "",
                 choices: [any] = None,
                 required: bool = True,
                 default_value: any = None,
                 validation_func: callable = None,
                 value: any = None,
                 supported_agents: [str] = None,
                 supported_agent_build_parameters: dict = None,
                 choice_filter_by_command_attributes: dict = None,
                 choices_are_all_commands: bool = False,
                 choices_are_loaded_commands: bool = False
                 dynamic_query_function: callable = None):
        self.name = name
        self.type = type
        self.description = description
        if choices is None:
            self.choices = []
        else:
            self.choices = choices
        self.required = required
        self.validation_func = validation_func
        if value is None:
            self.value = default_value
        else:
            self.value = value
        self.default_value = default_value
        self.supported_agents = supported_agents if supported_agents is not None else []
        self.supported_agent_build_parameters = supported_agent_build_parameters if supported_agent_build_parameters is not None else {}
        self.choice_filter_by_command_attributes = choice_filter_by_command_attributes if choice_filter_by_command_attributes is not None else {}
        self.choices_are_all_commands = choices_are_all_commands
        self.choices_are_loaded_commands = choices_are_loaded_commands
```

* `name` - the name of the parameter. This should match the name in the args dictionary
* `type`- this is the parameter type. The valid types are:
  * String&#x20;
  * Boolean&#x20;
  * File
  * Array
  * ChooseOne
  * ChooseMultiple&#x20;
  * Credential\_JSON
    * Get a JSON representation of all data for a credential
  * Credential\_Account
    * Get just the account name for a credential
  * Credential\_Realm
    * Get just the realm for a credential
  * Credential\_Type
    * Get just the credential type (i.e. hash, plaintext, key, ticket, etc)
  * Credential\_Value
    * Get just the actual credential value itself
  * Number&#x20;
  * Payload&#x20;
    * Select a payload that's already been generated and get the UUID for it. This is helpful for using that payload as a template to automatically generate another version of it to use as part of lateral movement or spawning new agents.
  * ConnectionInfo
    * Select the Host, Payload/Callback, and P2P profile for an agent or callback that you want to link to via a P2P mechanism. This allows you to generate random parameters for payloads (such as named-pipe names) and not require you to remember them when linking. You can simply select them and get all of that data passed to the agent.&#x20;
    * When this is up in the UI, you can also track new payloads on hosts in case Mythic isn't aware of them (maybe you moved and executed payloads in a method outside of Mythic). This allows Mythic to track that payload X is now on host Y and you can use the same selection process as the first bullet to filter down and select it for linking.&#x20;
  * LinkInfo
    * Get a list of all active/dead P2P connections for a given agent. Selecting one of these links gives you all the same information that you'd get from the `ConnectionInfo` parameter. The goal here is to allow you to easily select to "unlink" from an agent or to re-link to a very specific agent on a host that you were previously connected to.
* `description` - this is the description of the parameter that's presented to the user when the modal pops up for tasking
* `choices` - this is an array of choices if the type is `ChooseOne` or `ChooseMultiple`
  * If your command needs you to pick from the set of commands (rather than a static set of values), then there are a few other components that come into play. If you want the user to be able to select any command for this payload type, then set `choices_are_all_commands` to `True`.  Alternatively, you could specify that you only want the user to choose from commands that are already loaded into the callback, then you'd set `choices_are_loaded_commands` to `True`. As a modifier to either of these, you can set `choice_filter_by_command_attributes` to filter down the options presented to the user even more based on the parameters of the Command's `attributes` parameter. This would allow you to limit the user's list down to commands that are loaded into the current callback that support MacOS for example. An example of this would be:

```python
CommandParameter(name="test name", 
                 type=ParameterType.ChooseMultiple, 
                 description="so many choices!", 
                 choices_are_all_commands=True,
                 choice_filter_by_command_attributes={"supported_os": [SupportedOS.MacOS]}),
```

* `required` - this indicates if this is a required parameter or not. If the parameter is required and there's no default value or the user doesn't supply a value, an exception will be thrown
* `validation_func` - this is an additional function you can supply to do additional checks on values to make sure they're valid for the command. If a value isn't valid, an exception should be raised
* `value` - this is the final value for the parameter; it'll either be the default\_value or the value supplied by the user
* `default_value` - this is a value that'll be set if the user doesn't supply a value
* `supported_agents` - If your parameter type is `Payload` then you're expecting to choose from a list of already created payloads so that you can generate a new one. The `supported_agents` list allows you to narrow down that dropdown field for the user. For example, if you only want to see agents related to the `apfell` payload type in the dropdown for this parameter of your command, then set `supported_agents=["apfell"]` when declaring the parameter.
* `supported_agent_build_parameters` - allows you to get a bit more granular in specifying which agents you want to show up when you select the `Payload` parameter type. It might be the case that a command doesn't _just_ need instance of the `atlas` payload type, but maybe it only works with the `Atlas` payload type when it's compiled into .NET 3.5. This parameter value could then be `supported_agent_build_parameters={"atlas": {"version":"3.5"}}` . This value is a dictionary where the key is the name of the payload type and the value is a dictionary of what you want the build parameters to be.
* `dynamic_query_function` - More information can be found [here](dynamic-parameter-values.md), but you can provide a function here for ONLY parameters of type ChooseOne or ChooseMultiple where you dynamically generate the array of choices you want to provide the user when they try to issue a task of this type.

Most command parameters are pretty straight forward - the one that's a bit unique is the File type (where a user is uploading a file as part of the tasking). When you're doing your tasking, this `value` will be the base64 string of the file uploaded.&#x20;

![Viewing Command Information](<../../.gitbook/assets/Screen Shot 2020-07-14 at 3.47.37 PM.png>)

![Modal popup example in the UI when tasking](<../../.gitbook/assets/Screen Shot 2020-07-14 at 3.48.15 PM.png>)

## Example

Below is an example of a basic command and tasking for a `Cat` command. This command allows the user to go through the UI and supply a "path" or to do a short-hand and simply type `cat /path/to/file` on the tasking line. You can see this reflected in the `parse_arguments` function - checking to see if there's JSON supplied and if so, auto parse that json into the arguments, otherwise take what the user typed and set the path value to that.

```python
from mythic_payloadtype_container.MythicCommandBase import *
import json


class LsCommand(CommandBase):
    cmd = "ls"
    needs_admin = False
    help_cmd = "ls [directory]"
    description = "List directory."
    version = 1
    supported_ui_features = ["file_browser:list"]
    author = "@xorrior"
    argument_class = LsArguments
    attackmapping = ["T1083"]
    browser_script = BrowserScript(script_name="ls", author="@its_a_feature_")

    async def create_tasking(self, task: MythicTask) -> MythicTask:
        return task

    async def process_response(self, response: AgentResponse):
        pass
```

We'll talk about the `create_tasking` function next
