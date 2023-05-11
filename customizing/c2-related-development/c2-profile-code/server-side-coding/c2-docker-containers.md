# C2 Docker Containers

All C2 profiles are backed by a Docker container or intermediary layer of some sort.

## What's the goal of the container?

What do the C2 docker containers do? Why are things broken out this way? In order to make things more modular within Mythic, most services are broken out into their own containers. When it comes to C2 profiles, they simply serve as an intermediary layer that translates between your special sauce C2 mechanism and the normal RESTful interface that Mythic uses. This allows you to create any number of completely disjoint C2 types without having to modify anything in the main Mythic codebase.

### Container Information

C2 Profile Contianers, like Payload Type and Translation Containers, all use the same `mythic_base_container` Docker image. From there you can leverage the `github.com/MythicMeta/MythicContainer` GoLang package or the `mythic_container` PyPi package depending on if you want to write the meta information about your C2 profile in GoLang or Python. The actual code that binds to ports and accepts messages can be written in any language.

## Container Components

There are a few things needed to make a C2 container. For reference on general code structure options, check out [payload-type-development](../../../payload-type-development/ "mention"). The general structure is the same for Payload Types, C2 Profiles, and Translation containers.

Instead of declaring a new class of Payload Type though, you declare a new class of type C2Profile. For example, in Python you can do:

```python
from mythic_container.C2ProfileBase import *

class Websocket(C2Profile):
    name = "websocket"
    description = "Websocket C2 Server for poseidon"
    author = "@xorrior"
    is_p2p = False
    is_server_routed = False
    server_binary_path = pathlib.Path(".") / "websocket" / "c2_code" / "server"
    server_folder_path = pathlib.Path(".") / "websocket" / "c2_code"
```

The key here for a C2 Profile though is the `server_binary_path` - this indicates what actually gets executed to start listening for agent messages. This can be whatever you want, in any language you want, but you just need to make sure it's executable and identified here. If you want this to pick up something from the environment (and it's a script), be sure to put it as a `#!` at the top. For example, the default containers leverage python3, so they have `#! /usr/bin/env python3` at the top. This file is always executed via bash, so as a sub-process like `./server`

{% hint style="warning" %}
Your `server` code \*MUST\* send an HTTP header of `Mythic: ProfileNameHere` when connecting to Mythic. This allows Mythic to know _which_ profile is connecting
{% endhint %}

Within the `server_folder_path` should be a file called `config.json`, this is what the operator is able to edit through the UI and should contain all of the configuration components. The one piece that doesn't need to be here are if the operator needs to add additional files (like SSL certs).&#x20;

### C2Profile Class

This is a small class that just defines the metadata aspects of the C2 profile. A big piece here is the definition of the `parameters` array. Each element here is a `C2ProfileParameter` class instance with a few possible arguments:

```python
class ParameterType(Enum):
    String = "String"
    ChooseOne = "ChooseOne"
    Array = "Array"
    Date = "Date"
    Dictionary = "Dictionary"
    Boolean = "Boolean"


class DictionaryChoice:
    def __init__(self,
                 name: str,
                 default_value: str = "",
                 default_show: bool = True):
        self.name = name
        self.default_show = default_show
        self.default_value = default_value
    def to_json(self):
        return {
            "name": self.name,
            "default_value": self.default_value,
            "default_show": self.default_show
        }

class C2ProfileParameter:
    def __init__(
        self,
        name: str,
        description: str,
        default_value: any = None,
        randomize: bool = False,
        format_string: str = "",
        parameter_type: ParameterType = ParameterType.String,
        required: bool = True,
        verifier_regex: str = "",
        choices: list[str] = None,
        dictionary_choices: list[DictionaryChoice] = None,
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
        self.default_value = default_value
        self.crypto_type = crypto_type
        self.dictionary_choices = dictionary_choices

    def to_json(self):
        return {
            "name": self.name,
            "description": self.description,
            "default_value": self.default_value if self.parameter_type not in [ParameterType.Array, ParameterType.Dictionary] else json.dumps(self.default_value),
            "randomize": self.randomize,
            "format_string": self.format_string,
            "required": self.required,
            "parameter_type": self.parameter_type.value,
            "verifier_regex": self.verifier_regex,
            "crypto_type": self.crypto_type,
            "choices": self.choices,
            "dictionary_choices": [x.to_json() for x in self.dictionary_choices] if self.dictionary_choices is not None else None
        }
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
            name="headers",
            description="Customized headers",
            required=False,
            parameter_type=ParameterType.Dictionary,
            dictionary_choices=[
                DictionaryChoice(name="USER_AGENT",
                    default_value="Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko",
                    default_show=True,
                ),
                DictionaryChoice(name="host",
                    default_value="",
                    default_show=False,
                ),
            ]
        ),
```

This is saying that we have a Dictionary called "headers" with a few pre-set options.

<figure><img src="../../../../.gitbook/assets/Screenshot 2023-04-14 at 4.52.12 PM.png" alt=""><figcaption></figcaption></figure>



<figure><img src="../../../../.gitbook/assets/Screenshot 2023-04-14 at 4.52.38 PM.png" alt=""><figcaption></figcaption></figure>

