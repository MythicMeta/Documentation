# C2 Docker Containers

All C2 profiles are backed by a Docker container or intermediary layer of some sort.&#x20;

## What's the goal of the container?

What do the C2 docker containers do? Why are things broken out this way? In order to make things more modular within Mythic, most services are broken out into their own containers. When it comes to C2 profiles, they simply serve as an intermediary layer that translates between your special sauce C2 mechanism and the normal RESTful interface that Mythic uses. This allows you to create any number of completely disjoint C2 types without having to modify anything in the main Mythic codebase.

### Container Information

The most current docker container for C2 Profiles is `itsafeaturemythic/python38_sanic_c2profile:0.0.6`. This container, which has all of its source code hosted on [GitHub](https://github.com/MythicMeta/Mythic\_Docker\_Templates/tree/master/Docker\_C2\_Profile\_base\_files), uses the `mythic_c2_container` PyPi module to connect to Mythic and process requests. This PyPi module can be found on [GitHub](https://github.com/MythicMeta/Mythic\_C2\_Container), and is currently version 0.0.23 and reports to Mythic as version "4".&#x20;

## Container Components

There are a few things needed to make a C2 container. For this example, let's assume the name of the C2 profile is `ABC`:

1. Make the following folder structure by copying  `Example_C2_Profile` as `ABC` under `/Mythic/C2_Profiles/`.  The result should look like:

![](<../../../../.gitbook/assets/Screen Shot 2020-12-10 at 3.47.38 PM.png>)

2\. Within c2\_code should be a file called `server` (this is what will be executed when you select to start the c2 profile). If you want this to pick up something from the environment, be sure to put it as a `#!` at the top. For example, the default containers leverage python3, so they have `#! /usr/bin/env python3` at the top. This file is always executed via bash, so as a sub-process like `./server`

{% hint style="warning" %}
Your `server` code \*MUST\* send an HTTP header of `Mythic: ProfileNameHere` when connecting to Mythic. This allows Mythic to know _which_ profile is connecting
{% endhint %}

3\. Within c2\_code should be a file called `config.json`, this is what the operator is able to edit through the UI and should contain all of the configuration components. The one piece that doesn't need to be here are if the operator needs to add additional files (like SSL certs). There are too many security concerns and processes that go into arbitrary file upload currently that make this process require you to ssh into the host where Mythic is running and put the files there manually. This might change in the future, but that's the current expectation.

4\. Within `ABC` there should be a `Dockerfile` that sets up your environment. In most cases, you'll simply start it off with:

```
From itsafeaturemythic/python38_sanic_c2profile:0.0.6
```

But you can customize this as well

5\. Within the `mythic` folder are  all the basic files for processing tasking. The `rabbitmq_config.json` file is fine being left alone if you're doing a local Docker container. If you're turning another computer into your C2 container, similar to what you can do with Payload Types, then this will need to be modified in the same way as [Payload Type Development](../../../payload-type-development/#turning-a-vm-into-a-mythic-container).

6\. Within the `mythic` folder is the `c2_functions` folder. This is analogous to the `agent_functions`folder within Payload Types. This is where you'll put your Python files that define the C2 Profile metadata and establish any RPC functions you want to expose to other containers. This technically allows you to chain C2 containers together for RPC calls.

### C2Profile Class

Within the `/Mythic/C2_Profiles/ABC/mythic/c2_functions` folder is a python file (name doesn't matter, typically matches the c2 profile name though) with a new class that extends the base C2 Profile class. If you just copied from `Example_C2_Profile` though, then this will be called `HTTP.py`. An example would be:

```python
from mythic_c2_container.C2ProfileBase import *


class HTTP(C2Profile):
    name = "HTTP"
    description = "Uses HTTP(S) connections with a simple query parameter or basic POST messages. For more configuration options use dynamicHTTP."
    author = "@its_a_feature_"
    is_p2p = False
    is_server_routed = False
    parameters = [
        C2ProfileParameter(
            name="callback_port",
            description="Callback Port",
            default_value="80",
            verifier_regex="^[0-9]+$",
            required=False,
        ),
        C2ProfileParameter(
            name="killdate",
            description="Kill Date",
            parameter_type=ParameterType.Date,
            default_value=365,
            required=False,
        ),
        C2ProfileParameter(
            name="encrypted_exchange_check",
            description="Perform Key Exchange",
            choices=["T", "F"],
            parameter_type=ParameterType.ChooseOne,
        ),
        C2ProfileParameter(
            name="callback_jitter",
            description="Callback Jitter in percent",
            default_value="23",
            verifier_regex="^[0-9]+$",
            required=False,
        ),
        C2ProfileParameter(
            name="domain_front",
            description="Host header value for domain fronting",
            default_value="",
            required=False,
        ),
        C2ProfileParameter(
            name="USER_AGENT",
            description="User Agent",
            default_value="Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko",
            required=False,
        ),
        C2ProfileParameter(
            name="AESPSK",
            description="Crypto type",
            default_value="aes256_hmac",
            parameter_type=ParameterType.ChooseOne,
            choices=["aes256_hmac", "none"],
            required=False,
            crypto_type=True
        ),
        C2ProfileParameter(
            name="callback_host",
            description="Callback Host",
            default_value="https://domain.com",
            verifier_regex="^(http|https):\/\/[a-zA-Z0-9]+",
        ),
        C2ProfileParameter(
            name="get_uri",
            description="GET request URI",
            default_value="index",
            required=True,
        ),
        C2ProfileParameter(
            name="post_uri",
            description="POST request URI",
            default_value="data",
            required=True,
        ),
        C2ProfileParameter(
            name="query_path_name",
            description="Name of the query parameter",
            default_value="q",
            required=True,
        ),
        C2ProfileParameter(
            name="proxy_host",
            description="Proxy Host",
            default_value="",
            required=False,
            verifier_regex="^$|^(http|https):\/\/[a-zA-Z0-9]+",
        ),
        C2ProfileParameter(
            name="proxy_port",
            description="Proxy Port",
            default_value="",
            verifier_regex="^$|^[0-9]+$",
            required=False,
        ),
        C2ProfileParameter(
            name="proxy_user",
            description="Proxy Username",
            default_value="",
            required=False,
        ),
        C2ProfileParameter(
            name="proxy_pass",
            description="Proxy Password",
            default_value="",
            required=False,
        ),
        C2ProfileParameter(
            name="callback_interval",
            description="Callback Interval in seconds",
            default_value="10",
            verifier_regex="^[0-9]+$",
            required=False,
        ),
    ]
```

This is a small class that just defines the metadata aspects of the C2 profile. The main piece here is the definition of the `parameters` array. Each element here is a `C2ProfileParameter` class instance with a few possible arguments:

```python
class C2ProfileParameter:
    def __init__(
        self,
        name: str,
        description: str,
        default_value: str = "",
        randomize: bool = False,
        format_string: str = "",
        parameter_type: ParameterType = ParameterType.String,
        required: bool = True,
        verifier_regex: str = "",
        choices: [str] = None,
        crypto_type: bool = False,
    ):
        self.name = name
        self.description = description
        self.randomize = randomize
        self.format_string = format_string
        self.parameter_type = parameter_type
        self.required = required
        self.verifier_regex = verifier_regex
        self.choices = choices
        self.default_value = ""
        self.crypto_type = crypto_type
        if self.parameter_type == ParameterType.ChooseOne and choices is not None:
            self.default_value = "\n".join(choices)
        else:
            self.default_value = default_value
```

### C2 Profile Parameter Types

There are a few main values you can supply for `parameter_type` when defining the parameters of your c2 profile:

* `String` - This is simply a text box where you can input a string value
* `ChooseOne` - this is a dropdown choice where the operator makes a choice from a pre-defined list of options
* `Array` - This allows an operator to input an array of values rather than a single string value. When this is processed on the back-end, it becomes a proper array value like `["val1", "val2"]`.
* `Date` - This is a date in the YYYY-MM-DD format. However, when specifying a default value for this, you simply supply an _offset_ of the number of days from the current day. For example, to have a default value for the user be one year from the current date, the default\_value would be `365`. Similarly, you can supply negative values and it'll be days in the past. This manifests as a date-picker option for the user when generating a payload.
* `Dictionary` - This one is a bit more complicated, but allows you to specify an arbitrary dictionary for the user to generate and allows you to set some restrictions on the data. Let's take a look at this one more closely:

```python
C2ProfileParameter(
            name="USER_AGENT",
            description="User Agent",
            required=False,
            parameter_type=ParameterType.Dictionary,
            default_value=[
                {
                    "name": "USER_AGENT",
                    "max": 1,
                    "default_value": "Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko",
                    "default_show": True,
                },
                {
                    "name": "host",
                    "max": 2,
                    "default_value": "",
                    "default_show": False,
                },
                {
                    "name": "*",
                    "max": -1,
                    "default_value": "",
                    "default_show": False
                }
            ]
        ),
```

This is saying that we have a Dictionary called "USER\_AGENT" with a few pre-set options.&#x20;

* The first value is called "USER\_AGENT" and there can be at most one key with that name (max: 1). There's a default value for it and that one will be shown by default.
* There's another value called "host" that doesn't have a default value and allows you to specify 2 values for it (ex: this would be like a `{"host": ["val1, "val2"]}` situation. This one isn't shown by default, but you can add these in via the UI.
* There's one final entry with a name called "\*" and a max number of -1. This is a special scenario - `-1` means that there is no limit, you can have as many of these as you want. The name called "\*" is a special case where you get to write in your own key value. This allows you to give an operator the ability to supply arbitrary key-value pairs.

![Default view based on \`default\_show\` values](<../../../../.gitbook/assets/Screen Shot 2021-03-14 at 1.35.39 PM.png>)

![Adding more](<../../../../.gitbook/assets/Screen Shot 2021-03-14 at 1.39.50 PM.png>)

The two screenshots above show what this scenario looks like in the UI when generating a payload. The first one shows that by default, the user only sees the USER\_AGENT key with the default value, but they have the option to modify the value here or add more keys. Notice that an option is "host". The second screenshot shows that we were able to add two "host" keys, but now when we try to add more, "host" isn't an option and only "Custom" is. This matches up with our "\*" values that have an unlimited number (max: -1) and the host key having a max of 2.&#x20;
