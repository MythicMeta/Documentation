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
                                      dynamic_query_function=self.get_my_files)
        }

    async def get_my_files(self, callback: dict) -> [str]:
        resp = await MythicRPC().execute("get_file", callback_id=callback["id"], max_results=-1, filename=".*")
        return [r["filename"] for r in resp.response]

    async def parse_arguments(self):
        if len(self.command_line) > 0:
            if self.command_line[0] == "{":
                self.load_args_from_json_string(self.command_line)
            else:
                self.add_arg("command", self.command_line)
        else:
            raise ValueError("Missing arguments")
```

Here we can see that the `files` CommandParameter has an extra component - `dynamic_query_function`. This parameter points to a function that also lives within the same class, `get_my_files` in this case. This function is a little different than the other functions in the Command file because it occurs before you even have a task - this is generating parameters for when a user does a popup in the user interface. As such, this function gets one parameter - a dictionary of information about the callback itself. It should return an array of strings that will be presented to the user.&#x20;

{% hint style="info" %}
Dynamic queries are only supported for the ChooseOne and ChooseMultiple CommandParameter types
{% endhint %}

You have access to a lot of the same RPC functionality here that you do in `create_tasking`, but except for one notable exception - you don't have a task yet, so you have to do things based on the `callback_id`. You won't be able to create/delete entries via RPC calls, but you can still do pretty much every query capability. In this example, we're doing a `get_file` query to pull all files that exist within the current callback and present their filenames to the user.

## Accessible Info from Dynamic Function

What information do you have at your disposal during this dynamic function call? Via the dictionary parameter passed to your function (i.e. `callback` in the previous example), you have access to the following:

* build\_parameters - this key provides all of the build parameters used to generate the payload that this current callback generated from
* c2info -  this is an array of dictionaries of the different c2 profiles built into the callback. Just like [Payload Type Info](payload-type-info.md#building) where you iterate over them, you can do the same here. For example, to see a parameter called "callback\_host" in our first c2 profile is `callback.c2info[0]['callback_host']`.&#x20;
* agent\_callback\_id - id that agents see for the callback
* init\_callback - time of first checkin
* &#x20;last\_checkin = time of last checkin
* user = user context for the callback
* host = hostname of the callback
* pid = pid of the callback
* ip = internal ip of the callback
* external\_ip = external ip of the callback
* process\_name = name of the process that this callback is in
* description = description of the callback
* operator = operator that created the payload for this callback
* active = p.BooleanField(constraints=\[p.SQL("DEFAULT TRUE")], null=False)
* registered\_payload = UUID of the payload associated with this callback
* integrity\_level = integrity level of the callback (0 to 4)
* os = os value associated with the callback
* architecture =architecture of the callback
* domain = domain of the callback
* extra\_info = if the agent is storing extra information, it's here (this is a custom, free-form field for agents to use)
* sleep\_info = sleep information for the callback that agents can set
* id - the database ID of the callback that you can use in RPC calls
