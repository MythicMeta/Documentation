# Scripting Payloads

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.get_payloads()`:

```python
async def get_payloads(self) -> MythicResponse:
    """
    Get all the payloads for the current operation
    :return: a list of payload objects
    """

async def remove_payload(self, payload: Union[Payload, Dict]) -> MythicResponse:
    """
    Mark a payload as deleted in the database and remove it from disk
    Truly removing it from the database would delete any corresponding tasks/callbacks, so we don't do that
    :param payload:
    :return: a payload object
    """

async def create_payload(self, payload: Payload, all_commands: bool = None, timeout=None, wait_for_build: bool = None, exclude_commands: [str] = []) -> MythicResponse:
    """
    :param payload: A description of the payload you want
    :return: a payload object
    """

async def get_one_payload_info(self, payload: Union[Payload, Dict]) -> MythicResponse:
    """
    Get information about a specific payload
    :param payload:
        if using the Payload class, the following must be set:
            uuid
    :return: a Payload object
    """

async def download_payload(self, payload: Union[Payload, Dict]) -> bytes:
    """
    Get the final payload for a specified payload
    :param payload:
        if using Payload class, the following must be set:
            uuid
    :return: the content bytes of the payload
    """
```

This section also hooks into the following async callback functions:

```python
async def listen_for_all_payloads(self, callback_function=None, timeout=None):
        """
        Uses websockets to listen for all payloads within mythic for the current operation.
        :param callback_function: gets called on each notification
        :return:
        """
```

## Creating Payloads

Getting information for all existing payloads is one thing, and using that info to get detailed information on each payload is another thing, but being able to task Mythic to create new payloads is extremely useful. If you've created payloads in the browser UI though, you know there's a lot of different things that go into creating payloads. Scripting should be a time saver though, so you shouldn't _have_ to be extremely verbose. So, when creating payloads, simply describe what you'd like your payload to look like. Anything not explicitly set is interpreted by Mythic as a default value:

```python
p = mythic_rest.Payload(
    payload_type="poseidon", 
    c2_profiles={
        "http":[
                {"name": "callback_host", "value": "http://192.168.205.151"}
            ]
        },
    tag="test build",
    selected_os="macOS",
    build_parameters=[
        {
            "name": "mode", "value": "default"
        }],
    filename="whodidit.bin")
resp = await mythic.create_payload(p, all_commands=True, wait_for_build=True, exclude_commands=[])
await mythic_rest.json_print(resp)
```

In the above example we create our ideal payload. We set the `payload_type` to be the name of the payload type or it can be a PayloadType object. C2 profiles is a little trickier, but it's a dictionary of C2 profile name and an array of all the parameters. In this case, we're just using one C2 profile in the poseidon agent, the `HTTP` profile, and we're only setting the `callback_host` parameter. Since we're not specifying any of the other parameters, we're leaving them all to their default values. We specify the `tag` to give to our payload (this is what autopopulates the description for a callback), and we specify the `filename` for the payload.

If the payload takes build parameters (the extra, non-c2 profile pieces you see when you select an agent for creation in the UI), then you specify those in the `build_parameters` field. It's a list of dictionaries of `name` and `value` fields. In the above example, we set the `mode` build parameter for `poseidon` to `default` .&#x20;

{% hint style="info" %}
When selecting `all_commands=True` in the `create_payload` function, you will still be limited by the operating system you select. Additionally, if you want to select all commands _except_ for a few, you can supply a list of command names in the `exclude_commands` parameter to the `create_payload` call.&#x20;
{% endhint %}

### How do you find the "name" fields?

In the above snippet, you might be wondering where the name `callback_host` or `os` came from.&#x20;

For C2 profiles, you can see these specific value names if you change your `view_mode` for your operation from `operator` to `developer`. You'll see some additional features added on both the `Payload Types` page and the `C2 Profiles` page. On the C2 Profiles page, click the dropdown for `Configure` and select to `View Parameters`:

![](<../.gitbook/assets/Screen Shot 2020-08-25 at 12.56.52 PM.png>)

The far left-hand side where it has the `Name` column. That's the name you'll be referencing here. On the 3rd row you can see the `callback_host` name that we modified. From here, you can also see more of the "developer" based information such as which parameters are marked as `required` and which ones have specific regex requirements for what's a `valid` value.

For the build parameters on a payload type, go to the `Payload Types` page in the UI and select the dropdown for the corresponding payload type, then select `View Parameters`:

![](<../.gitbook/assets/Screen Shot 2020-08-25 at 1.28.05 PM.png>)

Again, we can see the `name` field on the far left is what we're interested in. In our example, we only set one build parameter - `os`. That means we used the default value for the other, `mode`, which in the case of a `ChooseOne` parameter type, will always be the first parameter listed.

### Specifying Commands for the Payload

You might have noticed so far that we didn't actually list any commands to be included in the agent. You can do one of two things:

1. specify a `commands=["cmd1", "cmd2", "cmd3"...]` argument to the Payload to specify which commands you want included in your agent
2. specify `all_commands=True` when actually tasking the creation, which will in turn cause the Scripting to pull information about the payload type you're trying to create, get the most recent listing of all the available commands, and automatically include those as part of your request. This is what we do for the poseidon example above.

### Submitting the Creation Request

Ok, so far you've just described the payload you want. Now you need to actually tell Mythic to create the thing you just described. This is pretty straight forward:

```python
resp = await mythic.create_payload(p, all_commands=True, wait_for_build=True)
await mythic_rest.json_print(resp)
```

The interesting piece here is the `wait_for_build` flag. So, everything in Mythic is separated out to different Docker containers, including the actual build pieces for agents. This means when you simply hit the RESTful endpoint to start a build, it returns pretty quickly, but that doesn't mean that the corresponding docker container is actually done building the agent. When you specify `wait_for_build=True`, after submitting the RESTful request, the script opens up a websocket connection to Mythic for information on the payload. It will then return when there's some sort of completion state for the payload (success or error). Your `resp` in this case will have the information about the payload in `resp.response`.&#x20;

## Downloading Payloads

So, you built a payload successfully or you see another payload that was previously built that you want to pull down for some purpose. That's pretty easy:

The `async def download_payload(self, payload: Union[Payload, Dict])` command allows you to specify the payload you want to download and returns to you the raw bytes of the payload.&#x20;

{% hint style="warning" %}
Unlike other commands, this does NOT return a MythicResponse object with the payload in the `.response` component. It simply returns the raw bytes to you.
{% endhint %}

