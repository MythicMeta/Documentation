# Dynamic Parameter Values

## What are dynamic parameters?

Sometimes when creating a command, the options you present to the operator might not always be static. For example, you might want to present them with a list of files that have been download; you might want to show a list of processes to choose from for injection; you might want to reach out to a remote service and display output from there. In all of these scenarios, the parameter choices for a user might change. Mythic can now support this.

## Where are dynamic parameters?

Since we're talking about a command's arguments, all of this lives in your Command's class that subclasses TaskArguments. Let's take an augmented `shell` example:

```python
class ShellArguments(TaskArguments):
    def __init__(self, command_line):
        super().__init__(command_line)
        self.args = {
            "command": CommandParameter(
                name="command", type=ParameterType.String, description="Command to run"
            ),
            "files": CommandParameter(name="files", type=ParameterType.ChooseOne, default_value=[],
                                      dynamic_query_function=self.get_files)
        }

    async def get_files(self, inputMsg: PTRPCDynamicQueryFunctionMessage) -> PTRPCDynamicQueryFunctionMessageResponse:
        fileResponse = PTRPCDynamicQueryFunctionMessageResponse(Success=False)
        file_resp = await SendMythicRPCFileSearch(MythicRPCFileSearchMessage(
            CallbackID=inputMsg.Callback,
            LimitByCallback=False,
            Filename="",
        ))
        if file_resp.Success:
            file_names = []
            for f in file_resp.Files:
                if f.Filename not in file_names and f.Filename.endswith(".exe"):
                    file_names.append(f.Filename)
            fileResponse.Success = True
            fileResponse.Choices = file_names
            return fileResponse
        else:
            fileResponse.Error = file_resp.Error
            return fileResponse

    async def parse_arguments(self):
        if len(self.command_line) > 0:
            if self.command_line[0] == "{":
                self.load_args_from_json_string(self.command_line)
            else:
                self.add_arg("command", self.command_line)
        else:
            raise ValueError("Missing arguments")
```

Here we can see that the `files` CommandParameter has an extra component - `dynamic_query_function`. This parameter points to a function that also lives within the same class, `get_files` in this case. This function is a little different than the other functions in the Command file because it occurs before you even have a task - this is generating parameters for when a user does a popup in the user interface. As such, this function gets one parameter - a dictionary of information about the callback itself. It should return an array of strings that will be presented to the user.

{% hint style="info" %}
Dynamic queries are only supported for the ChooseOne and ChooseMultiple CommandParameter types
{% endhint %}

You have access to a lot of the same RPC functionality here that you do in `create_tasking`, but except for one notable exception - you don't have a task yet, so you have to do things based on the `callback_id`. You won't be able to create/delete entries via RPC calls, but you can still do pretty much every query capability. In this example, we're doing a `get_file` query to pull all files that exist within the current callback and present their filenames to the user.

## Accessible Info from Dynamic Function

What information do you have at your disposal during this dynamic function call? Not much, but enough to do some RPC calls depending on the information you need to complete this function. Specifically, the PTRPCDynamicQueryFunctionMessage parameter has the following fields:

* command - name of the command
* parameter\_name - name of the parameter
* payload\_type - name of the payload type
* callback - the ID of the callback used for RPC calls

