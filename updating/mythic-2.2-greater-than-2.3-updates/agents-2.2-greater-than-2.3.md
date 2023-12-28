# Agents 2.2 -> 2.3

## PyPi Version

To update for Mythic version 2.3.7, you need to have the `mythic_payloadtype_container==0.1.7` PyPi version installed. This will report to Mythic as container version 12. After that, you'll need to perform the following updates so that your agent can successfully sync with Mythic and appear in the user interface. If you're using one of the `itsafeaturemythic` Docker images, check [this page](../../customizing/payload-type-development/payload-type-info/container-syncing.md#current-payloadtype-versions) for the appropriate Docker image to use.

## Build Parameters

Build parameters are moving from a dictionary format to an array. There's now also the build parameter type of `BuildParameterType.Boolean`.

```python
 build_parameters = [
        BuildParameter(
            name="mode",
            parameter_type=BuildParameterType.ChooseOne,
            description="Choose the build mode option. Select default for executables, "
                        "c-shared for a .dylib or .so file, "
                        "or c-archive for a .Zip containing C source code with an archive and header file",
            choices=["default", "c-archive", "c-shared"],
            default_value="default",
        ),
        BuildParameter(
            name="proxy_bypass",
            parameter_type=BuildParameterType.Boolean,
            default_value=False,
            description="Ignore HTTP proxy environment settings configured on the target host?",
        ),
    ]
```

There shouldn't have to be any other changes to this.

## Supported Operating Systems

In your payload definition file, you can specify the supported operating systems from a specific set of operating systems via `supported_os = [SupportedOS.Linux, SupportedOS.MacOS]`. To provide even more options going forward, if you want to specify an OS that doesn't have a pre-defined value (like `SupportedOS.Linux`), then you can supply your own via `SupportedOS("MyOS")`. At that point, `MyOS` would appear in the web interface as a build option.

## Browser Scripts

With the decommissioning of the old interface, we will also be decommissioning the old style of browser scripts. With that, there are no more `support_browser_scripts` in your payload definition file. That is strictly a relic of the old way of doing things.

In your individual commands though, there are still browser scripts. There needs to be an additional attribute `for_new_ui` added in to specify that a script is meant for the new UI vs the old UI.

```python
browser_script = [
  BrowserScript(script_name="ls", author="@its_a_feature_"),
  BrowserScript(script_name="ls_new", author="@its_a_feature_", for_new_ui=True)
]
```

More information on how to code browser scripts for the new UI can be found here: [#what-is-browser-scripting](../../customizing/payload-type-development/browser-scripting.md#what-is-browser-scripting "mention").

## Command Parameters / Arguments

There are a few things that changed as part of the `TaskArguments` processing. The first change is just with the `__init__` part - we're adding in a `**kwargs` parameter:

```python
class LsArguments(TaskArguments):
    def __init__(self, command_line, **kwargs):
        super().__init__(command_line, **kwargs)
```

So, you need to make sure you add in the `**kwargs` to _all_ of your command files. It's a little tedious, but it allows us to provide more and more features without you having to change stuff down the line. Today, the feature this provides is `tasking_location` - you'll be able to know _where_ tasking came from (more on this in a bit).

The next big piece is that the `self.args` dictionary is now an array, and the `CommandParameter` class has a few adjustments.

```python
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
```

So, as you can see from the above code block, you can just remove the dictionary pice and keep your `CommandParameter` as an array.

### CommandParameters

The `CommandParameter` class has a few slight adjustments. In addition to the `name` parameter, there's a `cli_name` and `display_name` parameter. This gives you the flexibility to refer to a parameter by `name` for your agent, by `cli_name` when a user is typing out parameters on the command line, and by `display_name` when the user opens up a tasking modal.

If you don't supply a `cli_name` then the `name` parameter will be used in its place. Similarly, if you don't supply a `display_name`, then `name` will be used. If `name` or `cli_name` have spaces, then the resulting `cli_name` will replace those spaces with `-` to make it more cli friendly.

The `required` and `ui_position` attributes have been removed! They are no longer part of the `CommandParameter` class. They are now part of the `ParameterGroupInfo` class.

#### parameter\_group\_info

To help with conditional parameters, Mythic 2.3 is introducing parameter groups. Every parameter must belong to at least one parameter group (if one isn't specified by you, then Mythic will add it to the `Default` group and make the parameter `required`.

You can specify this information via the `parameter_group_info` attribute on `CommandParameter` class. This attribute takes an array of `ParameterGroupInfo` objects. Each one of these objects has three attributes: `group_name` (string), `required`(boolean) `ui_position` (integer). These things together allow you to provide conditional parameter groups to a command.&#x20;

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
async def get_files(self, callback: dict) -> [str]:
        file_resp = await MythicRPC().execute("get_file", callback_id=callback["id"],
                                              limit_by_callback=False,
                                              get_contents=False,
                                              filename="",
                                              max_results=-1)
        if file_resp.status == MythicRPCStatus.Success:
            file_names = []
            for f in file_resp.response:
                if f["filename"] not in file_names:
                    file_names.append(f["filename"])
            return file_names
        else:
            return []
```

In the above code block, we're searching for files, not getting their contents, not limiting ourselves to just what's been uploaded to the callback we're tasking, and looking for all files (really it's all files that have "" in the name, which would be all of them). We then go through to de-dupe the filenames and return that list to the user.

#### Tasking Location and parse\_dictionary

Historically, we treated everything as just a `String` and passed everything the user typed, everything from a tasking modal, everything from scripting, etc to a single `parse_arguments` function. It was up to that function to then determine if it was looking at some form of JSON string or raw arguments or something else and parse it out into the actual CommandParameter objects.&#x20;

That's still the case, but only in some situations. Mythic now tracks _where_ tasking came from and can automatically handle certain instances for you. Mythic now tracks a `tasking_location` field which has the following values:

* `command_line` - this means that the input you're getting is just a raw string, like before. It could be something like `x86 13983 200` with a series of positional parameters for a command, it could be `{"command": "whoami"}` as a JSON string version of a dictionary of arguments, or anything else. In this case, Mythic really doesn't know enough about the source of the tasking or the contents of the tasking to provide more context.
* `parsed_cli` - this means that the input you're getting is a dictionary that was parsed by the new web interface's CLI parser. This is what happens when you type something on the command line for a command that has arguments (ex: `shell whoami` or `shell -command whoami`). Mythic can successfully parse out the parameters you've given into a single parameter\_group and gives you a `dictionary` of data.
* `modal` - this means that the input you're getting is a dictionary that came from the tasking modal. Nothing crazy here, but it does at least mean that there shouldn't be any silly shenanigans with potential parsing issues.&#x20;
* `browserscript` - if you click a tasking button from a browserscript table and that tasking button provides a dictionary to Mythic, then Mythic can forward that down as a dictionary. If the tasking button from a browserscript table submits a `String` instead, then that gets treated as `command_line` in terms of parsing.

With this ability to track where tasking is coming from and what form it's in, an agent's command file can choose to parse this data differently. By default, all commands must supply a `parse_arguments` function in their associated `TaskArguments` subclass. If you do nothing else, then _all_ of these various forms will get passed to that function as strings (if it's a dictionary it'll get converted into a JSON string). However, you can provide another function, `parse_dictionary` that can handle specifically the cases of parsing a given dictionary into the right CommandParameter objects as shown below:

```python
    async def parse_arguments(self):
        if len(self.command_line) == 0:
            raise ValueError("Must supply arguments")
        raise ValueError("Must supply named arguments or use the modal")

    async def parse_dictionary(self, dictionary_arguments):
        self.load_args_from_dictionary(dictionary_arguments
```

In this case, we are forcing the user to supply dictionary-based arguments from one of the methods above. The `self.load_args_from_dictionary(dictionary_arguments)` function takes in the dictionary supplied and looks through the keys to see if they match any `name` or `cli_name` parameters. The same sort of functionality is available in the `parse_arguments` function if you construct your own dictionary or you can call the `self.load_args_from_json_string(string_name_here)` to do the same thing but given a JSON string rather than a dictionary.

Inside of these functions you also have access to the `self.get_parameter_group_name()` function to get back the name of the matching parameter group based on which parameters have values. This function will either return a string value of the name or raise a `ValueError` exception with information about _why_ you don't currently match any group or why you match too many groups. You also have access to a `self.get_parameter_group_arguments` function which returns an array of just the `CommandParameters` that are associated with the group you're in.

