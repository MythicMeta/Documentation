# Commands

## What do Commands track?

Command information is tracked in your Payload Type's container. Each command has its own Python class or GoLang struct. In Python, you leverage `CommandBase` and `TaskArguments` to define information about the command and information about the command's arguments.

**CommandBase** defines the metadata about the command as well as any pre-processing functionality that takes place before the final command is ready for the agent to process. This class includes the `create_go_tasking` ([Create\_Tasking](../create\_tasking/)) and `process_response` ([Process Response](../process-response.md)) functions.

\*\*\*\*[**TaskArguments**](commands.md#taskarguments) does two things:

1. defines the parameters that the command needs
2. verifies / parses out the user supplied arguments into their proper components
   * this includes taking user supplied free-form input (like arguments to a sleep command - `10 4`) and parsing it into well-defined JSON that's easier for the agent to handle (like `{"interval": 10, "jitter": 4}`). This can also take user-supplied dictionary input and parse it out into the rightful CommandParameter objects.
   * This also includes verifying all the necessary pieces are present. Maybe your command requires a source and destination, but the user only supplied a source. This is where that would be determined and error out for the user. This prevents you from requiring your agent to do that sort of parsing in the agent.

If you're curious how this all plays out in a diagram, you can find one here: [#operator-submits-tasking](../../../message-flow/#operator-submits-tasking "mention").

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
    
    async def create_go_tasking(self, taskData: MythicCommandBase.PTTaskMessageAllData) -> MythicCommandBase.PTTaskCreateTaskingMessageResponse:
        response = MythicCommandBase.PTTaskCreateTaskingMessageResponse(
            TaskID=taskData.Task.ID,
            Success=True,
        )
        return response
        
    async def process_response(self, task: PTTaskMessageAllData, response: any) -> PTTaskProcessResponseMessageResponse:
        resp = PTTaskProcessResponseMessageResponse(TaskID=task.Task.ID, Success=True)
        return resp
```

Creating your own command requires extending this CommandBase class (i.e. `class ScreenshotCommand(CommandBase)` and providing values for all of the above components.

* `cmd` - this is the command name. The name of the class doesn't matter, it's this value that's used to look up the right command at tasking time
* `needs_admin` - this is a boolean indicator for if this command requires admin permissions
* `help_cmd` - this is the help information presented to the user if they type `help [command name]` from the main active callbacks page
* `description` - this is the description of the command. This is also presented to the user when they type help.
* `suported_ui_features` - This is an array of values that indicates where this command might be used within the UI. For example, from the active callbacks page, you see a table of all the callbacks. As part of this, there's a dropdown you can use to automatically issue an `exit` task to the callback. How does Mythic know which command to actually send? It's this array that dictates that. The following are used by the callback table, file browser, and process listing, but you're able to add in any that you want and leverage them via browser scripts for additional tasking:
  * supported\_ui\_features = \["callback\_table:exit"]
  * supported\_ui\_features = \["file\_browser:list"]
  * supported\_ui\_features = \["process\_browser:list"]
  * supported\_ui\_features = \["file\_browser:download"]
  * supported\_ui\_features = \["file\_browser:remove"]
  * supported\_ui\_features = \["file\_browser:upload"]
  * supported\_ui\_features = \["task\_response:interactive"]
* `version` - this is the version of the command you're creating/editing. This allows a helpful way to make sure your commands are up to date and tracking changes
* `argument_class` - this correlates this command to a specific `TaskArguments` class for processing/validating arguments
* `attackmapping` - this is a list of strings to indicate MITRE ATT\&CK mappings. These are in "T1113" format.
* `agent_code_path` is automatically populated for you like in building the payload. This allows you to access code files from within commands in case you need to access files, functions, or create new pieces of payloads. This is really useful for a `load` command so that you can find and read the functions you're wanting to load in.
* You can optionally add in the `attributes` variable. This is a new class called `CommandAttributes` where you can set whether or not your command supports being injected into a new process (some commands like `cd` or `exit` don't make sense for example). You can also provide a list of supported operating systems. This is helpful when you have a payload type that might compile into multiple different operating system types, but not all the commands work for all the possible operating systems. Instead of having to write "not implemented" or "not supported" function stubs, this will allow you to completely filter this capability out of the UI so users don't even see it as an option.
  * Available options are:
    * `supported_os` an array of SupportedOS fields (ex: `[SupportedOS.MacOS]`) (in Python for a new SupportedOS you can simply do `SupportedOS("my os name")`.
    * `spawn_and_injectable` is a boolean to indicate if the command can be injected into another process
    * `builtin` is a boolean to indicate if the command should be always included in the build process and can't be unselected
    * `load_only` is a boolean to indicate if the command can't be built in at the time of payload creation, but can be loaded in later
    * `suggested_command` is a boolean to indicate if the command should be pre-selected for users when building a payload
    * `filter_by_build_parameter` is a dictionary of `parameter_name:value` for what's required of the agent's build parameters. This is useful for when some commands are only available depending on certain values when building your agent (such as agent version).
    * You can also add in any other values you want for your own processing. These are simply `key=value` pairs of data that are stored. Some people use this to identify if a command has a dependency on another command. This data can be fetched via RPC calls for things like a `load` command to see what additional commands might need to be included.
  * This ties into the CommandParameter fields `choice_filter_by_command_attributes`, `choices_are_all_commands`, and `choices_are_loaded_commands`.
* The `create_go_tasking` function is very broad and covered in [Create\_Tasking](../create\_tasking/#create\_tasking)
* The `process_response` is similar, but allows you to specify that data shouldn't automatically be processed by Mythic when an agent checks in, but instead should be passed to this function for further processing and to use Mythic's RPC functionality to register the results into the system. The data passed here comes from the `post_response` message ([Process Response](../process-response.md)).
* The `script_only` flag indicates if this Command will be use strictly for things like issuing [subtasking](../sub-tasking-task-callbacks.md), but will NOT be compiled into the agent. The nice thing here is that you can now generate commands that don't need to be compiled into the agent for you to execute. These tasks never enter the "submitted" stage for an agent to pick up - instead they simply go into the [create\_tasking](../create\_tasking/) scenario (complete with subtasks and full RPC functionality) and then go into a completed state.

## TaskArguments

The TaskArguments class defines the arguments for a command and defines how to parse the user supplied string so that we can verify that all required arguments are supplied. Mythic now tracks _where_ tasking came from and can automatically handle certain instances for you. Mythic now tracks a `tasking_location` field which has the following values:

* `command_line` - this means that the input you're getting is just a raw string, like before. It could be something like `x86 13983 200` with a series of positional parameters for a command, it could be `{"command": "whoami"}` as a JSON string version of a dictionary of arguments, or anything else. In this case, Mythic really doesn't know enough about the source of the tasking or the contents of the tasking to provide more context.&#x20;

{% hint style="info" %}
When issuing tasks via Mythic's Scripting, they'll always come through as a tasking\_location of `command_line`.
{% endhint %}

* `parsed_cli` - this means that the input you're getting is a dictionary that was parsed by the new web interface's CLI parser. This is what happens when you type something on the command line for a command that has arguments (ex: `shell whoami` or `shell -command whoami`). Mythic can successfully parse out the parameters you've given into a single parameter\_group and gives you a `dictionary` of data.
* `modal` - this means that the input you're getting is a dictionary that came from the tasking modal. Nothing crazy here, but it does at least mean that there shouldn't be any silly shenanigans with potential parsing issues.&#x20;
* `browserscript` - if you click a tasking button from a browserscript table and that tasking button provides a dictionary to Mythic, then Mythic can forward that down as a dictionary. If the tasking button from a browserscript table submits a `String` instead, then that gets treated as `command_line` in terms of parsing.

With this ability to track where tasking is coming from and what form it's in, an agent's command file can choose to parse this data differently. By default, all commands must supply a `parse_arguments` function in their associated `TaskArguments` subclass. If you do nothing else, then _all_ of these various forms will get passed to that function as strings (if it's a dictionary it'll get converted into a JSON string). However, you can provide another function, `parse_dictionary` that can handle specifically the cases of parsing a given dictionary into the right CommandParameter objects as shown below:

```python
async def parse_arguments(self):
    if len(self.command_line) == 0:
        raise ValueError("Must supply arguments")
    if self.command_line[0] == "{":
        try:
            self.load_args_from_json_string(self.command_line)
            return
        except Exception as e:
            pass
    # if we got here, we weren't given a JSON string but raw text to parse
    # here's an example, though error prone because it splits on " " characters
    pieces = self.command_line.split(" ")
    self.add_arg("arg1", pieces[0])
    self.add_arg("arg2", pieces[1])

async def parse_dictionary(self, dictionary_arguments):
    self.load_args_from_dictionary(dictionary_arguments)
```

In `self.args` we define an array of our arguments and what they should be along with default values if none were provided.

In `parse_arguments` we parse the user supplied `self.command_line` into the appropriate arguments. The hard part comes when you allow the user to type arguments free-form and then must parse them out into the appropriate pieces.

```python
class LsArguments(TaskArguments):
    def __init__(self, command_line, **kwargs):
        super().__init__(command_line, **kwargs)
        self.args = [
            CommandParameter(
                name="path",
                type=ParameterType.String,
                default_value=".",
                description="Path of file or folder on the current system to list",
                parameter_group_info=[ParameterGroupInfo(
                    required=False
                )]
            )
        ]

    async def parse_arguments(self):
        self.add_arg("path", self.command_line)

    async def parse_dictionary(self, dictionary):
        if "host" in dictionary:
            # then this came from the file browser
            self.add_arg("path", dictionary["path"] + "/" + dictionary["file"])
            self.add_arg("file_browser", type=ParameterType.Boolean, value=True)
        else:
            self.load_args_from_dictionary(dictionary)
```

The main purpose of the TaskArguments class is to manage arguments for a command. It handles parsing the `command_line` string into `CommandParameters`, defining the `CommandParameters`, and providing an easy interface into updating/accessing/adding/removing arguments as needed.

As part of the `TaskArguments` subclass, you have access to the following pieces of information:

* `self.command_line` - the parameters sent down for you to parse
* `self.raw_command_line` - the original parameters that the user typed out. This is useful in case you have additional pieces of information to process or don't want information processed into the standard JSON/Dictionary format that Mythic uses.
* `self.tasking_location` - this indicates where the tasking came from
* `self.task_dictionary` - this is a dictionary representation of the task you're parsing the arguments for. You can see things like the initial `parameter_group_name` that Mythic parsed for this task, the user that issued the task, and more.
* `self.parameter_group_name` - this allows you to manually specify what the parameter group name should be. Maybe you don't want Mythic to do automatic parsing to determine the parameter group name, maybe you have additional pieces of data you're using to determine the group, or maybe you plan on adjusting it alter on. Whatever the case might be, if you set `self.parameter_group_name = "value"`, then Mythic won't continue trying to identify the parameter group based on the current parameters with values.

The class **must** implement the `parse_arguments` method and define the `args` array (it can be empty). This `parse_arguments` method is the one that allows users to supply "short hand" tasking and still parse out the parameters into the required JSON structured input. If you have defined command parameters though, the user can supply the required parameters on the command line (via `-commandParameterName` or via the popup tasking modal via `shift+enter`).

When syncing the command with the UI, Mythic goes through each class that extends the CommandBase, looks at the associated `argument_class`, and parses that class's `args` array of `CommandParameters` to create the pop-up in the UI.&#x20;

While the TaskArgument's `parse_arguments` method simply parses the user supplied input and sets the values for the named arguments, it's the CommandParameter's class that actually verifies that every required parameter has a value, that all the values are appropriate, and that default values are supplied if necessary.

### CommandParameters

CommandParameters, similar to BuildParameters, provide information for the user via the UI and validates that the values are all supplied and appropriate.

```python
class CommandParameter:
    def __init__(
        self,
        name: str,
        type: ParameterType,
        display_name: str = None,
        cli_name: str = None,
        description: str = "",
        choices: [any] = None,
        default_value: any = None,
        validation_func: callable = None,
        value: any = None,
        supported_agents: [str] = None,
        supported_agent_build_parameters: dict = None,
        choice_filter_by_command_attributes: dict = None,
        choices_are_all_commands: bool = False,
        choices_are_loaded_commands: bool = False,
        dynamic_query_function: callable = None,
        parameter_group_info: [ParameterGroupInfo] = None
    ):
        self.name = name
        if display_name is None:
            self.display_name = name
        else:
            self.display_name = display_name
        if cli_name is None:
            self.cli_name = name 
        else:
            self.cli_name = cli_name
        self.type = type
        self.user_supplied = False # keep track of if this is using the default value or not
        self.description = description
        if choices is None:
            self.choices = []
        else:
            self.choices = choices
        self.validation_func = validation_func
        if value is None:
            self._value = default_value
        else:
            self.value = value
        self.default_value = default_value
        self.supported_agents = supported_agents if supported_agents is not None else []
        self.supported_agent_build_parameters = supported_agent_build_parameters if supported_agent_build_parameters is not None else {}
        self.choice_filter_by_command_attributes = choice_filter_by_command_attributes if choice_filter_by_command_attributes is not None else {}
        self.choices_are_all_commands = choices_are_all_commands
        self.choices_are_loaded_commands = choices_are_loaded_commands
        self.dynamic_query_function = dynamic_query_function
        if not callable(dynamic_query_function) and dynamic_query_function is not None:
            raise Exception("dynamic_query_function is not callable")
        self.parameter_group_info = parameter_group_info
        if self.parameter_group_info is None:
            self.parameter_group_info = [ParameterGroupInfo()]
```

* `name` - the name of the parameter that your agent will use. `cli_name` is an optional variation that you want user's to type when typing out commands on the command line, and `display_name` is yet another optional name to use when displaying the parameter in a popup tasking modal.
* `type`- this is the parameter type. The valid types are:
  * String - gets a string value
  * Boolean - gets a boolean value
  * File
    * Upload a file through your browser. In your create tasking though, you get a String UUID of the file that can be used via SendMythicRPC\* calls to get more information about the file or the file contents
  * Array
    * An Array of string values
  * TypedArray
    * An array of arrays, ex: `[ ["int": "5"], ["char*", "testing"] ]`
  * ChooseOne - gets a string value
  * ChooseMultiple
    * An Array of string values
  * Credential\_JSON
    * Select a specific credential that's registered in the Mythic credential store. In your create tasking, get a JSON representation of all data for that credential
  * Number&#x20;
  * Payload
    * Select a payload that's already been generated and get the UUID for it. This is helpful for using that payload as a template to automatically generate another version of it to use as part of lateral movement or spawning new agents.
  * ConnectionInfo
    * Select the Host, Payload/Callback, and P2P profile for an agent or callback that you want to link to via a P2P mechanism. This allows you to generate random parameters for payloads (such as named-pipe names) and not require you to remember them when linking. You can simply select them and get all of that data passed to the agent.
    * When this is up in the UI, you can also track new payloads on hosts in case Mythic isn't aware of them (maybe you moved and executed payloads in a method outside of Mythic). This allows Mythic to track that payload X is now on host Y and you can use the same selection process as the first bullet to filter down and select it for linking.
  * LinkInfo
    * Get a list of all active/dead P2P connections for a given agent. Selecting one of these links gives you all the same information that you'd get from the `ConnectionInfo` parameter. The goal here is to allow you to easily select to "unlink" from an agent or to re-link to a very specific agent on a host that you were previously connected to.
* `description` - this is the description of the parameter that's presented to the user when the modal pops up for tasking
* `choices` - this is an array of choices if the type is `ChooseOne` or `ChooseMultiple`
  * If your command needs you to pick from the set of commands (rather than a static set of values), then there are a few other components that come into play. If you want the user to be able to select any command for this payload type, then set `choices_are_all_commands` to `True`. Alternatively, you could specify that you only want the user to choose from commands that are already loaded into the callback, then you'd set `choices_are_loaded_commands` to `True`. As a modifier to either of these, you can set `choice_filter_by_command_attributes` to filter down the options presented to the user even more based on the parameters of the Command's `attributes` parameter. This would allow you to limit the user's list down to commands that are loaded into the current callback that support MacOS for example. An example of this would be:

```python
CommandParameter(name="test name", 
                 type=ParameterType.ChooseMultiple, 
                 description="so many choices!", 
                 choices_are_all_commands=True,
                 choice_filter_by_command_attributes={"supported_os": [SupportedOS.MacOS]}),
```

* `choices` - for the `TypedArray` type, the `choices` here is the list of options you want to provide in the dropdown for the user. So if you have choices as `["int", "char*"]`, then when the user adds a new array entry in the modal, those two will be the options. Additionally, if you set the `default_value` to `char*`, then `char*` will be the value selected by default.
* `validation_func` - this is an additional function you can supply to do additional checks on values to make sure they're valid for the command. If a value isn't valid, an exception should be raised
* `value` - this is the final value for the parameter; it'll either be the default\_value or the value supplied by the user. This isn't something you set directly.
* `default_value` - this is a value that'll be set if the user doesn't supply a value
* `supported_agents` - If your parameter type is `Payload` then you're expecting to choose from a list of already created payloads so that you can generate a new one. The `supported_agents` list allows you to narrow down that dropdown field for the user. For example, if you only want to see agents related to the `apfell` payload type in the dropdown for this parameter of your command, then set `supported_agents=["apfell"]` when declaring the parameter.
* `supported_agent_build_parameters` - allows you to get a bit more granular in specifying which agents you want to show up when you select the `Payload` parameter type. It might be the case that a command doesn't _just_ need instance of the `atlas` payload type, but maybe it only works with the `Atlas` payload type when it's compiled into .NET 3.5. This parameter value could then be `supported_agent_build_parameters={"atlas": {"version":"3.5"}}` . This value is a dictionary where the key is the name of the payload type and the value is a dictionary of what you want the build parameters to be.
* `dynamic_query_function` - More information can be found [here](../dynamic-parameter-values.md), but you can provide a function here for ONLY parameters of type ChooseOne or ChooseMultiple where you dynamically generate the array of choices you want to provide the user when they try to issue a task of this type.
* `typedarray_parse_function` - This allows you to have typed arrays more easily displayed and parsed throughout Mythic (useful for BOF/COFF work). More information for this can be found [here](../12-typedarray-parse-function.md).

Most command parameters are pretty straight forward - the one that's a bit unique is the File type (where a user is uploading a file as part of the tasking). When you're doing your tasking, this `value` will be the base64 string of the file uploaded.

### parameter\_group\_info

To help with conditional parameters, Mythic 2.3 introduced parameter groups. Every parameter must belong to at least one parameter group (if one isn't specified by you, then Mythic will add it to the `Default` group and make the parameter `required`).

You can specify this information via the `parameter_group_info` attribute on `CommandParameter` class. This attribute takes an array of `ParameterGroupInfo` objects. Each one of these objects has three attributes: `group_name` (string), `required`(boolean) `ui_position` (integer). These things together allow you to provide conditional parameter groups to a command.&#x20;

{% hint style="info" %}
A note about **required**: This indicates if you require a value from the user. If you can provide a sane default\_value for a parameter, then it isn't _required_. Your agent might need a value, but if the `default_value` works, then it isn't _required_ as far as Mythic is concerned. The `required` attribute here tells Mythic that if a user didn't explicitly provide a parameter, then it needs to open up the dialog modal to ask them to provide one. For example: the `path` parameter for listing a directory might not be required because if one isn't provided by the user, you can assume to list the contents of the current working directory. However, the `path` parameter for something like `download` would be required because if the user just typed `download` on the command line, you'd have no sane default value to use instead.
{% endhint %}

Let's look at an example - the new `apfell` agent's `upload` command now leverages conditional parameters. This command allows you to either:

* specify a `remote_path` and a `filename` - Mythic then looks up the filename to see if it's already been uploaded to Mythic before. If it has, Mythic can simply use the same file identifier and pass that along to the agent.
* specify a `remote_path` and a `file` - This is uploading a new file, registering it within Mythic, and then passing along that new file identifier

Notice how both options require the `remote_path` parameter, but the `file` and `filename` parameters are mutually exclusive.

```python
class UploadArguments(TaskArguments):
    def __init__(self, command_line, **kwargs):
        super().__init__(command_line, **kwargs)
        self.args = [
            CommandParameter(
                name="file", cli_name="new-file", display_name="File to upload", type=ParameterType.File, description="Select new file to upload",
                parameter_group_info=[
                    ParameterGroupInfo(
                        required=True,
                        group_name="Default"
                    )
                ]
            ),
            CommandParameter(
                name="filename", cli_name="registered-filename", display_name="Filename within Mythic", description="Supply existing filename in Mythic to upload",
                type=ParameterType.ChooseOne,
                dynamic_query_function=self.get_files,
                parameter_group_info=[
                    ParameterGroupInfo(
                        required=True,
                        group_name="specify already uploaded file by name"
                    )
                ]
            ),
            CommandParameter(
                name="remote_path",
                cli_name="remote_path",
                display_name="Upload path (with filename)",
                type=ParameterType.String,
                description="Provide the path where the file will go (include new filename as well)",
                parameter_group_info=[
                    ParameterGroupInfo(
                        required=True,
                        group_name="Default",
                        ui_position=1
                    ),
                    ParameterGroupInfo(
                        required=True,
                        group_name="specify already uploaded file by name",
                        ui_position=1
                    )
                ]
            ),
        ]
```

So, the `file` parameter has one `ParameterGroupInfo` that calls out the parameter as required. The `filename` parameter also has one `ParameterGroupInfo` that calls out the parameter as required. It also has a `dynamic_query_function` that allows the task modal to run a function to populate the selection box. Lastly, the `remote_path` parameter has TWO `ParameterGroupInfo` objects in its array - one for each group. This is because the `remote_path` parameter applies to both groups. You can also see that we have a `ui_position` specified for these which means that regardless of which option you're viewing in the tasking modal, the parameter `remote_path` will be the first parameter shown. This helps make things a bit more consistent for the user.

If you're curious, the function used to get the list of files for the user to select is here:

```python
async def get_files(self, inputMsg: PTRPCDynamicQueryFunctionMessage) -> PTRPCDynamicQueryFunctionMessageResponse:
        fileResponse = PTRPCDynamicQueryFunctionMessageResponse(Success=False)
        file_resp = await MythicRPC().execute("get_file", callback_id=inputMsg.Callback,
                                              limit_by_callback=False,
                                              filename="",
                                              max_results=-1)
        if file_resp.status == MythicRPCStatus.Success:
            file_names = []
            for f in file_resp.response:
                if f["filename"] not in file_names and f["filename"].endswith(".exe"):
                    file_names.append(f["filename"])
            fileResponse.Success = True
            fileResponse.Choices = file_names
            return fileResponse
        else:
            fileResponse.Error = file_resp.error
            return fileResponse
```

In the above code block, we're searching for files, not getting their contents, not limiting ourselves to just what's been uploaded to the callback we're tasking, and looking for all files (really it's all files that have "" in the name, which would be all of them). We then go through to de-dupe the filenames and return that list to the user.

### Processing Order

So, with all that's going on, it's helpful to know what gets called, when, and what you can do about it.
