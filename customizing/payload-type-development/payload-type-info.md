# Payload Type Definition

## 1.0 Payload Type Definition

Payload Type information must be set and pulled from a definition either in Python or in GoLang. Below are basic examples in Python and GoLang:

{% tabs %}
{% tab title="Python" %}
```python
from mythic_container.PayloadBuilder import *
from mythic_container.MythicCommandBase import *
from mythic_container.MythicRPC import *
import json


class Apfell(PayloadType):
    name = "apfell"
    file_extension = "js"
    author = "@its_a_feature_"
    supported_os = [SupportedOS.MacOS]
    wrapper = False
    wrapped_payloads = []
    note = """This payload uses JavaScript for Automation (JXA) for execution on macOS boxes."""
    supports_dynamic_loading = True
    c2_profiles = ["http", "dynamichttp"]
    mythic_encrypts = True
    translation_container = None # "myPythonTranslation"
    build_parameters = []
    agent_path = pathlib.Path(".") / "apfell"
    agent_icon_path = agent_path / "agent_functions" / "apfell.svg"
    agent_code_path = agent_path / "agent_code"

    build_steps = [
        BuildStep(step_name="Gathering Files", step_description="Making sure all commands have backing files on disk"),
        BuildStep(step_name="Configuring", step_description="Stamping in configuration values")
    ]

    async def build(self) -> BuildResponse:
        # this function gets called to create an instance of your payload
        resp = BuildResponse(status=BuildStatus.Success)
        return resp
        
```

There are a couple key pieces of information here:

* Line 1-2 imports all of basic classes needed for creating an agent
* line 7 defines the new class (our agent). This can be called whatever you want, but the important piece is that it extends the `PayloadType` class as shown with the `()`.
* Lines 8-27 defines the parameters for the payload type that you'd see throughout the UI.
  * the name is the name of the payload type
  * supported\_os is an array of supported OS versions
  * supports\_dynamic\_loading indicates if the agent allows you to select only a subset of commands when creating an agent or not
  * build\_parameters is an array describing all of the build parameters when creating your agent
  * c2\_profiles is an array of c2 profile names that the agent supports
* Line 18 defines the name of a "translation container" which we will talk about in another section, but this allows you to support your own, non-mythic message format, custom crypto, etc.
* The last piece is the function that's called to **build** the agent based on all of the information the user provides from the web UI.

The `PayloadType` base class is in the `PayloadBuilder.py` file. This is an abstract class, so your instance needs to provide values for all these fields.
{% endtab %}

{% tab title="Golang" %}
```go
package agentfunctions

import (
	"bytes"
	"encoding/json"
	"fmt"
	agentstructs "github.com/MythicMeta/MythicContainer/agent_structs"
	"github.com/MythicMeta/MythicContainer/mythicrpc"
	"os"
	"os/exec"
	"path/filepath"
	"strings"
)

var payloadDefinition = agentstructs.PayloadType{
	Name:                                   "basicAgent",
	FileExtension:                          "bin",
	Author:                                 "@xorrior, @djhohnstein, @Ne0nd0g, @its_a_feature_",
	SupportedOS:                            []string{agentstructs.SUPPORTED_OS_LINUX, agentstructs.SUPPORTED_OS_MACOS},
	Wrapper:                                false,
	CanBeWrappedByTheFollowingPayloadTypes: []string{},
	SupportsDynamicLoading:                 false,
	Description:                            "A fully featured macOS and Linux Golang agent",
	SupportedC2Profiles:                    []string{"http", "websocket", "poseidon_tcp"},
	MythicEncryptsData:                     true,
	BuildParameters: []agentstructs.BuildParameter{
		{
			Name:          "mode",
			Description:   "Choose the build mode option. Select default for executables, c-shared for a .dylib or .so file, or c-archive for a .Zip containing C source code with an archive and header file",
			Required:      false,
			DefaultValue:  "default",
			Choices:       []string{"default", "c-archive", "c-shared"},
			ParameterType: agentstructs.BUILD_PARAMETER_TYPE_CHOOSE_ONE,
		},
		{
			Name:          "architecture",
			Description:   "Choose the agent's architecture",
			Required:      false,
			DefaultValue:  "AMD_x64",
			Choices:       []string{"AMD_x64", "ARM_x64"},
			ParameterType: agentstructs.BUILD_PARAMETER_TYPE_CHOOSE_ONE,
		},
		{
			Name:          "proxy_bypass",
			Description:   "Ignore HTTP proxy environment settings configured on the target host?",
			Required:      false,
			DefaultValue:  false,
			ParameterType: agentstructs.BUILD_PARAMETER_TYPE_BOOLEAN,
		},
		{
			Name:          "garble",
			Description:   "Use Garble to obfuscate the output Go executable.\nWARNING - This significantly slows the agent build time.",
			Required:      false,
			DefaultValue:  false,
			ParameterType: agentstructs.BUILD_PARAMETER_TYPE_BOOLEAN,
		},
	},
	BuildSteps: []agentstructs.BuildStep{
		{
			Name:        "Configuring",
			Description: "Cleaning up configuration values and generating the golang build command",
		},

		{
			Name:        "Compiling",
			Description: "Compiling the golang agent (maybe with obfuscation via garble)",
		},
	},
}

func build(payloadBuildMsg agentstructs.PayloadBuildMessage) agentstructs.PayloadBuildResponse {
	payloadBuildResponse := agentstructs.PayloadBuildResponse{
		PayloadUUID:        payloadBuildMsg.PayloadUUID,
		Success:            true,
		UpdatedCommandList: &payloadBuildMsg.CommandList,
	}
	return payloadBuildResponse
}
```
{% endtab %}
{% endtabs %}

### 1.1 Wrapper Payloads

A quick note about wrapper payload types - there's only a few differences between a wrapper payload type and a normal payload type. A configuration variable, `wrapper`, determines if something is a wrapper or not. A wrapper payload type takes as input the output of a previous build (normal payload type or wrapper payload type) along with build parameters and generates a new payload. A wrapper payload type does NOT have any c2 profiles associated with it because it's simply wrapping an existing payload.&#x20;

An easy example is thinking of the `service_wrapper` - this wrapper payload type takes in the shellcode version of another payload and "wraps" it in the execution of a service so that it'll properly respond to the service control manager on windows. A similar example would be to take an agent and wrap it in an MSBuild format. These things don't have their own C2, but rather just package/wrap an existing agent into a new, more generic, format.

{% tabs %}
{% tab title="Python" %}
To access the payload that you're going to wrap, use the `self.wrapped_payload` attribute during your `build` execution. This will be the base64 encoded version of the payload you're going to wrap.
{% endtab %}

{% tab title="Golang" %}
To access the payload that you're going to wrap, use the `payloadBuildMsg.WrappedPayload` attribute during your `build` execution. This will be the raw bytes of the payload you're going to wrap.
{% endtab %}
{% endtabs %}

When you're done generating the payload, you'll return your new result the exact same way as normal payloads (as part of the [build](payload-type-info.md#building) process).

## 2.0 Build Parameters

Build parameters define the components shown to the user when creating a payload.

![](<../../.gitbook/assets/Screen Shot 2021-12-02 at 11.38.16 AM.png>)

The `BuildParameter` class has a couple of pieces of information that you can use to customize and validate the parameters supplied to your build:

{% tabs %}
{% tab title="Python" %}
The most up-to-date code is available in the `https://github.com/MythicMeta/MythicContainerPyPi` repository.

```python
class BuildParameterType(str, Enum):
    """Types of parameters available for building payloads

    Attributes:
        String:
            A string value
        ChooseOne:
            A list of choices for the user to select exactly one
        ChooseMultiple:
            A list of choices for the user to select 0 or more
        Array:
            The user can supply multiple values in an Array format
        Date:
            The user can select a Date in YYYY-MM-DD format
        Dictionary:
            The user can supply a dictionary of values
        Boolean:
            The user can toggle a switch for True/False
        File:
            The user can select a file that gets uploaded - a file UUID gets passed in during build
        TypedArray:
            The user can supply an array where each element also has a drop-down option of choices
    """
    String = "String"
    ChooseOne = "ChooseOne"
    ChooseMultiple = "ChooseMultiple"
    Array = "Array"
    Date = "Date"
    Dictionary = "Dictionary"
    Boolean = "Boolean"
    File = "File"
    TypedArray = "TypedArray"
```
{% endtab %}

{% tab title="Golang" %}
The most up-to-date code is available at `https://github.com/MythicMeta/MythicContainer`.

```go
type BuildParameterType = string

const (
   BUILD_PARAMETER_TYPE_STRING          BuildParameterType = "String"
   BUILD_PARAMETER_TYPE_BOOLEAN                            = "Boolean"
   BUILD_PARAMETER_TYPE_CHOOSE_ONE                         = "ChooseOne"
   BUILD_PARAMETER_TYPE_CHOOSE_MULTIPLE                    = "ChooseMultiple"
   BUILD_PARAMETER_TYPE_DATE                               = "Date"
   BUILD_PARAMETER_TYPE_DICTIONARY                         = "Dictionary"
   BUILD_PARAMETER_TYPE_ARRAY                              = "Array"
   BUILD_PARAMETER_TYPE_NUMBER                             = "Number"
   BUILD_PARAMETER_TYPE_FILE                               = "File"
   BUILD_PARAMETER_TYPE_TYPED_ARRAY                        = "TypedArray"
)

// BuildParameter - A structure defining the metadata about a build parameter for the user to select when building a payload.
type BuildParameter struct {
   // Name - the name of the build parameter for use during the Payload Type's build function
   Name string `json:"name"`
   // Description - the description of the build parameter to be presented to the user during build
   Description string `json:"description"`
   // Required - indicate if this requires the user to supply a value or not
   Required bool `json:"required"`
   // VerifierRegex - if the user is supplying text and it needs to match a specific pattern, specify a regex pattern here and the UI will indicate to the user if the value is valid or not
   VerifierRegex string `json:"verifier_regex"`
   // DefaultValue - A default value to show the user when building in the Mythic UI. The type here depends on the Parameter Type - ex: for a String, supply a string. For an array, provide an array
   DefaultValue interface{} `json:"default_value"`
   // ParameterType - The type of parameter this is so that the UI can properly render components for the user to modify
   ParameterType BuildParameterType `json:"parameter_type"`
   // FormatString - If Randomize is true, this regex format string is used to generate a value when presenting the option to the user
   FormatString string `json:"format_string"`
   // Randomize - Should this value be randomized each time it's shown to the user so that each payload has a different value
   Randomize bool `json:"randomize"`
   // IsCryptoType -If this is True, then the value supplied by the user is for determining the _kind_ of crypto keys to generate (if any) and the resulting stored value in the database is a dictionary composed of the user's selected and an enc_key and dec_key value
   IsCryptoType bool `json:"crypto_type"`
   // Choices - If the ParameterType is ChooseOne or ChooseMultiple, then the options presented to the user are here.
   Choices []string `json:"choices"`
   // DictionaryChoices - if the ParameterType is Dictionary, then the dictionary choices/preconfigured data is set here
   DictionaryChoices []BuildParameterDictionary `json:"dictionary_choices"`
}

// BuildStep - Identification of a step in the build process that's shown to the user to eventually collect start/end time as well as stdout/stderr per step
type BuildStep struct {
   Name        string `json:"step_name"`
   Description string `json:"step_description"`
}
```
{% endtab %}
{% endtabs %}

* `name` is the name of the parameter, if you don't provide a longer description, then this is what's presented to the user when building your payload
* `parameter_type` describes what is presented to the user - valid types are:
  * `BuildParameterType.String`
    * During build, this is a string
  * `BuildParameterType.ChooseOne`
    * During build, this is a string
  * `BuildParameterType.ChooseMultiple`
    * During build, this is an array of strings
  * `BuildParameterType.Array`
    * During build, this is an array of strings
  * `BuildParameterType.Date`
    * During build, this is a string of the format `YYYY-MM-DD`
  * `BuildParameterType.Dictionary`
    * During build, this is a dictionary
  * `BuildParameterType.Boolean`
    * During build, this is a boolean
  * `BuildParameterType.File`
    * During build, this is a string UUID of the file (so that you can use a MythicRPC call to fetch the contents of the file)
  * `BuildParameterType.TypedArray`
    * During build, this is an arrray of arrays, always in the format `[ [ type, value], [type value], [type, value] ...]`
* `required` indicates if there must be a value supplied. If no value is supplied by the user and no default value supplied here, then an exception is thrown before execution gets to the `build` function.&#x20;
* `verifier_regex` is a regex the web UI can use to provide some information to the user about if they're providing a valid value or not
* `default_value` is the default value used for building if the user doesn't supply anything
* `choices` is where you can supply an array of options for the user to pick from if the parameter\_type is ChooseOne
* `dictionary_choice`s are the choices and metadata about what to display to the user for key-value pairs that the user might need to supply
* `value` is the component you access when building your payload - this is the final value (either the default value or the value the user supplied)
* `verifier_func` is a function you can provide for additional checks on the value the user supplies to make sure it's what you want. This function should either return nothing or raise an exception if something isn't right

As a recap, where does this come into play? In the first section, we showed a section like:

{% tabs %}
{% tab title="Python" %}
```python
build_parameters = [
        BuildParameter(name="string", parameter_type=BuildParameterType.String, 
                       description="test string", default_value="test"),
        BuildParameter(name="choose one", parameter_type=BuildParameterType.ChooseOne, 
                       description="test choose one",
                       choices=["a", "b"], default_value="a"),
        BuildParameter(name="choose one crypto", 
                       parameter_type=BuildParameterType.ChooseOne, 
                       description="choose one crypto",
                       crypto_type=True, choices=["aes256_hmac", "none"]),
        BuildParameter(name="date", parameter_type=BuildParameterType.Date, 
                       default_value=30, description="test date offset from today"),
        BuildParameter(name="array", parameter_type=BuildParameterType.Array, 
                       default_value=["a", "b"],
                       description="test array"),
        BuildParameter(name="dict", parameter_type=BuildParameterType.Dictionary, 
                       dictionary_choices=[
            DictionaryChoice(name="host", default_value="abc.com", default_show=False),
            DictionaryChoice(name="user-agent", default_show=True, default_value="mozilla")
        ],
                       description="test dictionary"),
        BuildParameter(name="random", parameter_type=BuildParameterType.String, 
                       randomize=True, format_string="[a,b,c]{3}",
                       description="test randomized string"),
        BuildParameter(name="bool", parameter_type=BuildParameterType.Boolean, 
                       default_value=True, description="test boolean")
    ]
    
```
{% endtab %}

{% tab title="Golang" %}
```go
BuildParameters: []agentstructs.BuildParameter{
		{
			Name:          "mode",
			Description:   "Choose the build mode option. Select default for executables, c-shared for a .dylib or .so file, or c-archive for a .Zip containing C source code with an archive and header file",
			Required:      false,
			DefaultValue:  "default",
			Choices:       []string{"default", "c-archive", "c-shared"},
			ParameterType: agentstructs.BUILD_PARAMETER_TYPE_CHOOSE_ONE,
		},
		{
			Name:          "architecture",
			Description:   "Choose the agent's architecture",
			Required:      false,
			DefaultValue:  "AMD_x64",
			Choices:       []string{"AMD_x64", "ARM_x64"},
			ParameterType: agentstructs.BUILD_PARAMETER_TYPE_CHOOSE_ONE,
		},
		{
			Name:          "proxy_bypass",
			Description:   "Ignore HTTP proxy environment settings configured on the target host?",
			Required:      false,
			DefaultValue:  false,
			ParameterType: agentstructs.BUILD_PARAMETER_TYPE_BOOLEAN,
		},
		{
			Name:          "garble",
			Description:   "Use Garble to obfuscate the output Go executable.\nWARNING - This significantly slows the agent build time.",
			Required:      false,
			DefaultValue:  false,
			ParameterType: agentstructs.BUILD_PARAMETER_TYPE_BOOLEAN,
		},
	},
```
{% endtab %}
{% endtabs %}

## 3.0 Building

You have to implement the `build` function and return an instance of the `BuildResponse` class. This response has these fields:

* `status` - an instance of BuildStatus (Success or Error)
  * Specifically, `BuildStatus.Success` or `BuildStatus.Error`
* `payload` - the raw bytes of the finished payload (if you failed to build, set this to `None` or empty bytes like `b''` in Python.
* `build_message` - any stdout data you want the user to see
* `build_stderr` - any stderr data you want the user to see
* `build_stdout` - any stdout data you want the user to see
* `updated_filename` - if you want to update the filename to something more appropriate, set it here.&#x20;
  * For example: the user supplied a filename of `apollo.exe` but based on the build parameters, you're actually generating a dll, so you can update the filename to be `apollo.dll`. This is particularly useful if you're optionally returning a zip of information so that the user doesn't have to change the filename before downloading. If you plan on doing this to update the filename for a wide variety of options, then it might be best to leave the file extension field in your payload type definition blank `""` so that you can more easily adjust the extension.

The most basic version of the build function would be:

{% tabs %}
{% tab title="Python" %}
```python
async def build(self) -> BuildResponse:
        # this function gets called to create an instance of your payload
        return BuildResponse(status=BuildStatus.Success)
```
{% endtab %}

{% tab title="Golang" %}
```go
func build(payloadBuildMsg agentstructs.PayloadBuildMessage) agentstructs.PayloadBuildResponse {
	payloadBuildResponse := agentstructs.PayloadBuildResponse{
		PayloadUUID:        payloadBuildMsg.PayloadUUID,
		Success:            true,
	}
	return payloadBuildResponse
}
```
{% endtab %}
{% endtabs %}

Once the `build` function is called, all of your `BuildParameters` will already be verified (all parameters marked as `required` will have a `value` of some form (user supplied or default\_value) and all of the verifier functions will be called if they exist). This allows you to _know_ that by the time your `build` function is called that all of your parameters are valid.

Your build function gets a few pieces of information to help you build the agent (other than the build parameters):

From within your build function, you'll have access to the following pieces of information:

{% tabs %}
{% tab title="Python" %}
* `self.uuid` - the UUID associated with your payload
  * This is how your payload identifies itself to Mythic before getting a new Staging and final Callback UUID
* `self.commands` - a wrapper class around the names of all the commands the user selected.
  * Access this list via `self.commands.get_commands()`

```python
for cmd in self.commands.get_commands():
    command_code += open(self.agent_code_path / "{}.js".format(cmd), 'r').read() + "\n"
```

* `self.agent_code_path` - a `pathlib.Path` object pointing to the path of the `agent_code` directory that holds all the code for your payload. This is something you pre-define as part of your agent definition.
  * To access "test.js" in that "agent\_code" folder, simply do:\
    `f = open(self.agent_code_path / "test.js", 'r')`.
  * With `pathlib.Path` objects, the `/` operator allows you to concatenate paths in an OS agnostic manner. This is the recommended way to access files so that your code can work anywhere.
* `self.get_parameter("parameter name here")`
  * The build parameters that are validated from the user. If you have a build\_parameter with a name of "version", you can access the user supplied or default value with `self.get_parameter("version")`
* `self.selected_os` - This is the OS that was selected on the first step of creating a payload
* `self.c2info` - this holds a list of dictionaries of the c2 parameters and c2 class information supplied by the user. This is a list because the user can select multiple c2 profiles (maybe they want HTTP and SMB in the payload for example). For each element in self.c2info, you can access the information about the c2 profile with `get_c2profile()` and access to the parameters via `get_parameters_dict()`. Both of these return a dictionary of key-value pairs.
  * the dictionary returned by `self.c2info[0].get_c2profile()` contains the following:
    * `name` - name of the c2 profile
    * `description` - description of the profile
    * `is_p2p` - boolean of if the profile is marked as a p2p profile or not
  * the dictionary returned by `self.c2info[0].get_parameters_dict()`contains the following:
    * `key` - value
      * where each `key` is the `key` value defined for the c2 profile's parameters and `value` is what the user supplied. You might be wondering where to get these keys? Well, it's not too crazy and you can view them right in the UI - [Name Fields](broken-reference).
      * If the C2 parameter has a value of `crypto_type=True`, then the "value" here will be a bit more than just a string that the user supplied. Instead, it'll be a dictionary with three pieces of information: `value` - the value that the user supplied, `enc_key` - a base64 string (or None) of the encryption key to be used, `dec_key` - a base64 string (or None) of the decryption key to be used. This gives you more flexibility in automatically generating encryption/decryption keys and supporting crypto types/schemas that Mythic isn't aware of. In the HTTP profile, the key `AESPSK` has this type set to True, so you'd expect that dictionary.
      * If the C2 parameter has a type of "Dictionary", then things are a little different.&#x20;
        * Let's take the "headers" parameter in the `http` profile for example. This allows you to set header values for your `http` traffic such as User-Agent, Host, and more. When you get this value on the agent side, you get an array of values that look like the following:\
          `{"User-Agent": "the user agent the user supplied", "MyCustomHeader": "my custom value"}`. You get the final "dictionary" that's created from the user supplied fields.
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
            if key == "AESPSK":
                c2_code = c2_code.replace(key, val["enc_key"] if val["enc_key"] is not None else "")
            elif not isinstance(val, str):
                c2_code = c2_code.replace(key, json.dumps(val))
            else:
                c2_code = c2_code.replace(key, val)
    except Exception as p:
        build_msg += str(p)
        pass
```
{% endtab %}

{% tab title="Golang" %}
```go
// PayloadBuildMessage - A structure of the build information the user provided to generate an instance of the payload type.
// This information gets passed to your payload type's build function.
type PayloadBuildMessage struct {
	// PayloadType - the name of the payload type for the build
	PayloadType string `json:"payload_type" mapstructure:"payload_type"`
	// Filename - the name of the file the user originally supplied for this build
	Filename string `json:"filename" mapstructure:"filename"`
	// CommandList - the list of commands the user selected to include in the build
	CommandList []string `json:"commands" mapstructure:"commands"`
	// build param name : build value
	// BuildParameters - map of param name -> build value from the user for the build parameters defined
	// File type build parameters are supplied as a string UUID to use with MythicRPC for fetching file contents
	// Array type build parameters are supplied as []string{}
	BuildParameters PayloadBuildArguments `json:"build_parameters" mapstructure:"build_parameters"`
	// C2Profiles - list of C2 profiles selected to include in the payload and their associated parameters
	C2Profiles []PayloadBuildC2Profile `json:"c2profiles" mapstructure:"c2profiles"`
	// WrappedPayload - bytes of the wrapped payload if one exists
	WrappedPayload *[]byte `json:"wrapped_payload,omitempty" mapstructure:"wrapped_payload"`
	// WrappedPayloadUUID - the UUID of the wrapped payload if one exists
	WrappedPayloadUUID *string `json:"wrapped_payload_uuid,omitempty" mapstructure:"wrapped_payload_uuid"`
	// SelectedOS - the operating system the user selected when building the agent
	SelectedOS string `json:"selected_os" mapstructure:"selected_os"`
	// PayloadUUID - the Mythic generated UUID for this payload instance
	PayloadUUID string `json:"uuid" mapstructure:"uuid"`
	// PayloadFileUUID - The Mythic generated File UUID associated with this payload
	PayloadFileUUID string `json:"payload_file_uuid" mapstructure:"payload_file_uuid"`
}

// PayloadBuildC2Profile - A structure of the selected C2 Profile information the user selected to build into a payload.
type PayloadBuildC2Profile struct {
	Name  string `json:"name" mapstructure:"name"`
	IsP2P bool   `json:"is_p2p" mapstructure:"is_p2p"`
	// parameter name: parameter value
	// Parameters - this is an interface of parameter name -> parameter value from the associated C2 profile.
	// The types for the various parameter names can be found by looking at the build parameters in the Mythic UI.
	Parameters map[string]interface{} `json:"parameters" mapstructure:"parameters"`
}

type CryptoArg struct {
	Value  string `json:"value" mapstructure:"value"`
	EncKey string `json:"enc_key" mapstructure:"enc_key"`
	DecKey string `json:"dec_key" mapstructure:"dec_key"`
}
```
{% endtab %}
{% endtabs %}

Finally, when building a payload, it can often be helpful to have both stdout and stderr information captured, especially if you're compiling code. Because of this, you can set the `build_message` ,`build_stderr` , and `build_stdout` fields of the `BuildResponse` to have this data. For example:



{% tabs %}
{% tab title="Python" %}
```python
    async def build(self) -> BuildResponse:
        # this function gets called to create an instance of your payload
        resp = BuildResponse(status=BuildStatus.Success)
        # create the payload
        build_msg = ""

        #create_payload = await MythicRPC().execute("create_callback", payload_uuid=self.uuid, c2_profile="http")
        try:
            command_code = ""
            for cmd in self.commands.get_commands():
                try:
                    command_code += (
                            open(self.agent_code_path / "{}.js".format(cmd), "r").read() + "\n"
                    )
                except Exception as p:
                    pass
            base_code = open(
                self.agent_code_path / "base" / "apfell-jxa.js", "r"
            ).read()
            await SendMythicRPCPayloadUpdatebuildStep(MythicRPCPayloadUpdateBuildStepMessage(
                PayloadUUID=self.uuid,
                StepName="Gathering Files",
                StepStdout="Found all files for payload",
                StepSuccess=True
            ))
            base_code = base_code.replace("UUID_HERE", self.uuid)
            base_code = base_code.replace("COMMANDS_HERE", command_code)
            all_c2_code = ""
            if len(self.c2info) != 1:
                resp.build_stderr = "Apfell only supports one C2 Profile at a time"
                resp.set_status(BuildStatus.Error)
                return resp
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
                        if key == "AESPSK":
                            c2_code = c2_code.replace(key, val["enc_key"] if val["enc_key"] is not None else "")
                        elif not isinstance(val, str):
                            c2_code = c2_code.replace(key, json.dumps(val))
                        else:
                            c2_code = c2_code.replace(key, val)
                except Exception as p:
                    build_msg += str(p)
                    pass
                all_c2_code += c2_code
            base_code = base_code.replace("C2PROFILE_HERE", all_c2_code)
            await SendMythicRPCPayloadUpdatebuildStep(MythicRPCPayloadUpdateBuildStepMessage(
                PayloadUUID=self.uuid,
                StepName="Configuring",
                StepStdout="Stamped in all of the fields",
                StepSuccess=True
            ))
            resp.payload = base_code.encode()
            if build_msg != "":
                resp.build_stderr = build_msg
                resp.set_status(BuildStatus.Error)
            else:
                resp.build_message = "Successfully built!\n"
        except Exception as e:
            resp.set_status(BuildStatus.Error)
            resp.build_stderr = "Error building payload: " + str(e)
        return resp
```
{% endtab %}

{% tab title="Golang" %}
{% @github-files/github-code-block %}
{% endtab %}
{% endtabs %}

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

### Build Steps

The last thing to mention are build steps. These are defined as part of the agent and are simply descriptions of what is happening during your build process. The above example makes some RPC calls for `SendMythicRPCPayloadUpdatebuildStep` to update the build steps back to Mythic while the build process is happening. For something as fast as the `apfell` agent, it'll appear as though all of these happen at the same time. For something that's more computationally intensive though, it's helpful to provide information back to the user about what's going on - stamping in values? obfuscating? compiling? more obfuscation? opsec checks? etc. Whatever it is that's going on, you can provide this data back to the operator complete with stdout and stderr.

## Execution flow

So, what's the actual, end-to-end execution flow that goes on? A diagram can be found here: [#what-happens-for-building-payloads](../../message-flow/#what-happens-for-building-payloads "mention").

1. PayloadType container is started, it connects to Mythic and sends over its data (by parsing all these python files)
2. An operator wants to create a payload from it, so they go to "Create Components" -> "Create Payload"
3. The operator selects an OS type that the agent supports (ex. Linux, macOS, Windows)
4. The operator selects all c2 profiles they want included
   1. and for each c2 selected, provides any c2 required parameters
5. The operator selects this payload type
6. The operator fills out/selects all of the payload type's build parameters
7. The operator selects all commands they want included in the payload
8. Mythic takes all of this information and sends it to the payload type container
9. The container sends the `BuildResponse` message back to the Mythic server.
