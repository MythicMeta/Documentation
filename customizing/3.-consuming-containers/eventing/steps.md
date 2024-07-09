# Steps

## What are they?

An eventgroup workflow has a trigger that causes the entire workflow to start. Once it starts, it's the individual steps that perform actions. Once a steps is complete, the next step(s) start based on which other steps they depend on. Each step has the following:

* `name` - string - a unique name within the eventgroup workflow
* `description` - string - a description of what the step is for
* `depends_on` - array of strings - a list of other step names that this step needs to complete first before this step can start
* `action` - string - the specific action to take for this step
* `action_data` - dictionary - per-action specific data that is necessary for that step (ex: think payload config information for creating a payload or command name and parameters for issuing a task).
* `environment` - dictionary - per-step unique environment information you want available during step execution
* `inputs` - dictionary - the inputs that are going into the step that come from things outside of your knowledge when you first upload the workflow.
* `outputs` - dictionary - the data you want to export from this step so that it's available to be used as part of the `inputs` of another step
* `continue_on_error` - a boolean - indicates if you want to continue onto the next step if this step fails. Normally, if a step hits an error case then the entire workflow is cancelled from then on.

## inputs

Each step can define inputs that are necessary for the step that may or may not be known at the time that the eventgroup workflow is uploaded to Mythic. The point of this information is that you want to use it to replace sections of your `action_data` based on other factors. A simple example is that you have a workflow that triggers on new callbacks, and you want to issue a new task to that callback. Well, in order for you to issue a task, you need to know _which_ callback was just created.&#x20;

Inputs are a dictionary where the key is the value that's swapped out in the `action_data` or made available in actions like `custom_function` and the value indicates where the data comes from. Let's take an example to make that clearer:

### inputs example

```yaml
name: "whoami on new callbacks"
description: "automatically issue whoami on new callbacks"
trigger: callback_new
trigger_data:
  payload_types:
    - poseidon
keywords:
  - poseidon_callback

steps:
  - name: "issue whoami"
    description:
    inputs:
      CALLBACK_ID: env.display_id
      API: mythic.apitoken
      COMMAND: shell
      RUBEUS_FILE_ID: upload.Rubeus.exe
    action: task_create
    action_data:
      callback_display_id: CALLBACK_ID
      params: whoami
      command_name: COMMAND
```

In the above example we have an eventgroup workflow called `whoami on new callbacks`, which, as you might expect, wants to `automatically issue whoami on new callbacks`. This is triggered based on `callback_new` , but limited to only the `poseidon` payload types. This can also be triggered via `keywords` using the `poseidon_callback` keyword (more on that later).  This workflow has one step called `issue whoami` that takes 4 inputs, has an action of `task_create`, and has some `action_data`.&#x20;

#### input value keywords

The inputs dictionary has keys on the left (here they're all caps, but it doesn't matter) and on the right are some specially formatted strings. You'll notice that most are in the form `X.Y`. Mythic checks these and determines if the `X` is a special keyword. The possible keywords are:

* `env` - fetch the named value from the environment (including information that triggered the workflow). In the above example, we fetch `display_id` from the callback information that triggered the workflow and store it in the `CALLBACK_ID` variable for use in `action_data.`
  * If you're ever curious what data is automatically available to you in this field, you can trigger the event, then right click one of the steps and click "View Details". The first dropdown, "Original and Instance Metadata", will show inputs, outputs, and environment data.
* `upload` - this allows you to specify the name of a file that has been uploaded to Mythic. The resulting value is the `agent_file_id` UUID value for that file or `""` if it doesn't exist.
* `download` - this allows you to specify the name of a file that was downloaded to Mythic. The resulting value is the `agent_file_id` UUID value for that file, or `""` if it doesn't exist.
* `workflow` - this allows you to specify the name of a file that was uploaded as part of the workflow. You can see these files and upload/remove them by clicking the paperclip icon in the `actions` column when viewing the eventgroup workflow.
* `mythic` - this allows you to get various pieces of information from Mythic that you can't get elsewhere. This will expand over time, but currently the only options for after the `.` are the following:
  * `apitoken` - this will generate an APIToken that you can use for this step to interact with Mythic's GraphQL scripting. This APIToken will be invalid after the step completes, so plan accordingly. Any actions taken with this APIToken are tracked and available by right clicking an instance of a step that uses this and viewing details.
* **anything else** - if your input is of the format `X.Y`, but doesn't match one of the above keywords, then Mythic checks if `X` matches the name of a step. If so, then it looks for an `output` key in that step that matches `Y`. If that exists, then Mythic will swap it out. If either of those conditions aren't met though, then just the static valued is used.

After going through to find the values for the various `inputs` fields, the corresponding data is replaced within the `action_data` - for example, `CALLBACK_ID` from our `inputs` key matches the `CALLBACK_ID` in the `action_data`, so it gets swapped out. Same with `COMMAND`.&#x20;

## Actions

*   `payload_create` - This action allows you to start building a payload. This action is over once the payload finishes (success or error).

    * `action_data` - a dictionary of the data used to create a new payload. This is the same sort of data when you click to "export" a payload's configuration on the payloads page.
      * `description` - the description for the new payload
      * `payload_type` - the name of the payload type for the payload
      * `selected_os` - the name of the selected operating system
      * `filename` - the name of the file you want at the end
      * `wrapped_payload` - if you're wrapping another payload, specify the wrapped payload's UUID here
      * `c2_profiles` - an array of c2 profile data which is as follows:
        * `c2_profile` - the name of the c2 profile
        * `c2_profile_parameters` - a dictionary of the parameters for the c2 profile in key-value format
      * `build_parameters` - an array of build parameter dictionary values as follows:
        * `name` - the name of the build parameter
        * `value` - the value of the build parameter

    ex:

    ```yaml
    steps:
      - name: "apollo bin"
        description: "generate shellcode"
        action: "payload_create"
        action_data:
          payload_type: "apollo"
          description: "apollo test payload shellcode"
          selected_os: "Windows"
          build_parameters:
          - name: "output_type"
            value: "Shellcode"
          filename: "apollo.bin"
          c2_profiles:
          - c2_profile: "websocket"
            c2_profile_parameters:
              AESPSK: "aes256_hmac"
              callback_host: "ws://192.168.0.118"
              tasking_type: "Push"
          commands:
          - shell
          - exit
          - load
        outputs:
          PayloadUUID: "uuid"
        environment:
      - name: "apollo service"
        description: "service exe with apollo shellcode"
        action: "payload_create"
        inputs:
          WRAPPER_UUID: "apollo bin.PayloadUUID"
        depends_on:
        - "bin opsec checker"
        action_data:
          payload_type: "service_wrapper"
          description: "apollo service exe"
          selected_os: "Windows"
          build_parameters:
          - name: "version"
            value: "4.0"
          - name: "arch"
            value: "x64"
          filename: "apollo_service.exe"
          wrapped_payload: WRAPPER_UUID
        outputs:
          PayloadUUID: "uuid"
    ```
* `callback_create` - This action allows you to create a new callback.
  * `action_data` - a dictionary of data used to create a new callback:
    * `payload_uuid` - the payload UUID that was used to create this callback
    * `c2_profile` - the name of the c2 profile that was used to "create" this callback
    * `encryption_key` - base64 bytes of an optional encryption key to use
    * `decryption_key` - base64 bytes of an optional decryption key to use
    * `crypto_type` - string type of crypto to use (just like you'd select from the dropdown menu when generating a payload) if you want to use something other than what was selected for your payload/c2 profile.
    * `user` - the username for the callback context
    * `host` - the hostname of the computer where the callback is executing
    * `pid` - the pid of the callback process
    * `extra_info` - a string of any extra information you want to save/store about this callback
    * `sleep_info` - a string of extra information you want to save/store about the sleep context for this callback (interval, jitter, skew, etc)
    * `ip` - the ip of the callback (if you only collect one)
    * `ips` - an array of ip strings
    * `external_ip` - if you know your external ip and want to supply it
    * `os` - specific os information about the host
    * `domain` - the domain name associated with the host
    * `architecture` - the architecture of the process
    * `description` - a custom description for the callback
    * `process_name` - the name of the binary that's backing your process execution
*   `task_create` - This action allows you to create a new task. This action is over once the task finishes (success or error).

    * `action_data` - a dictionary of the data used to create a new task:
      * `callback_display_id` - the display id of the callback to task
      * `command_name` - the name of the command you want to issue
      * `payload_type` - if you're leveraging a command\_augment container and the command name isn't unique, then specify which payload\_type you are referring to here.
      * `params` - if you just have a string of parameters to supply, you can provide that here
      * `param_dictionary` - if your command optionally supports providing structured data, provide a dictionary here of your parameter names -> values
      * `parameter_group_name` - if you are using parameter groups and want to explicitly say which one to use, put that here
      * `token` - if you're using a token associated with the callback, specify that here. If you don't put the token id then you won't use a token at all
      * `parent_task_id` - if you're wanting to issue a task as a subtask of another task, specify the ID of the parent task here
      * `is_interactive_task` - if the task you're issuing is part of an ongoing interactive task, specify that here (and be sure to specify the `parent_task_id` too, which would be the one that started the interactive tasking session)
      * `interactive_task_type` - if this is an interactive task, specify what kind of task input it is as an int value

    ```yaml
    action_data:
          callback_display_id: CALLBACK_ID
          params: whoami
          command_name: COMMAND
    ```
* `custom_function` - This action allows you to execute a custom function within a custom `event` container that you create or install. The benefit here is that you can get access to the entirety of Mythic's Scripting and GraphQL API by using an input of `mythic.apitoken` . This allows you to do almost anything.
  * `action_data` - dictionary with the following two things. As part of the function execution, you get access to the environment, inputs, and action data, so there's plenty of ways to get additional context and information for your custom function.
    * `container_name` - the name of the container that has the custom function you want to execute
    * `function_name` - the name of the function you want to execute within that container
* `conditional_check` - This action allows you to run a custom function within a custom `event` container that you create or install with the purpose of identifying if certain steps should be skipped or not. Normally, if a step hits an error then the entire workflow is cancelled, but you can use this `conditional_check` to run custom code to determine if potentially problematic steps should be skipped or not.
  * `action_data` - dictionary with the following three things. As part of the function execution, you get access tot he environment, inputs, and action data, so there's plenty of ways to get additional context and information for your custom function.
    * `container_name` - the name of the container that has the custom conditional check you want to execute
    * `function_name` - the name of the conditional check you want to execute within that container
    * `steps` - an array of step names that you want to potentially skip based on the result of this function call
* `task_intercept` - This allows you to intercept a task after the task's `opsec_post` function finishes to have one final opportunity to block the task from executing. This can only be used in conjunction with the `task_intercept` trigger. You can have additional steps as part of the workflow, but one step must be `task_intercept` if you have a trigger of `task_intercept`.
  * `action_data` - dictionary with the following. You get access to the environment, inputs, action data, and taskid so that you can fetch more information about the task and use GraphQL to perform any additional actions you might need.
    * `container_name` - the name of the container that has the task intercept function you want to execute. There can only be one per container, so you don't need to specify a function name.&#x20;
* `response_intercept` - This allows you to intercept the `user_output` response of an agent before it goes to the Mythic UI for an operator. Just like with `task_intercept`, this must exist if you have a trigger of `response_intercept` and can't be used if that's not the trigger. This allows you to get access to the output that the agent returned, then modify it before forwarding it along to Mythic. This means you can modify the response (add context, change values, etc) before the user ever sees it. Tasks with output that's been intercepted will have a special symbol next to them in the UI.
  * `action_data` - a dictionary with the following. You get access to the environment, inputs, action data, and response id so that you can fetch more information about the response (and thus task) to use with GraphQL to perform any additional actions you might need.
    * `container_name` - the name of the container that has the response intercept function you want to execute. There can only be one per container, so you don't need to specify a function name

## outputs

Outputs from a step allow you to expose something from one step to another step. Say for example that you have a step that creates a new payload, but you want to use that new payload's UUID as input for the next step that creates a wrapper around that payload. You need some way to expose that specific UUID (there might be multiple if you're creating multiple payloads). This is where outputs come into play. We've already seen from inputs in that last example, **anything else**, where prior steps' outputs can be examined to look for values to use as inputs to new steps.

When Mythic processes outputs for a step, it loops through certain data depending on the action:

* `payload_create` - Currently, the following are available as keyword outputs:
  * `uuid` - this returns the UUID of the payload that was created
  * `build_phase` - this returns the string build phase of the payload that was created
  * `id` - this returns the ID of the payload that was created
* `task_create` - Currently, the following are available as keyword outputs:
  * `status` - the string status of the task
  * `params` - the string parameters that were passed down to the agent
  * `original_params` - the original parameters that were supplied as a string
  * `command_name` - the name of the command that was sent down to the agent (if any)
  * `display_id` - the display id integer of the task that was created
  * `token` - the token id associated with the task if any
  * `parent_task_id` - the id of the parent task associated with this task (if any)
  * `agent_task_id` - the UUID of the task that the agent would see
* `callback_create` - Currently, the following are available as keyword outputs:
  * `display_id` - the display ID of the callback that was created
  * `agent_callback_id` - the UUID of the callback that was created that agents see
  * `id` - the int ID of the callback that was created

Everything else that's returned by a step in "outputs" is unmodified. Let's take an example:

```yaml
steps:
  - name: "apollo bin"
    description: "generate shellcode"
    action: "payload_create"
    action_data:
      payload_type: "apollo"
      description: "apollo test payload shellcode"
      selected_os: "Windows"
      build_parameters:
      - name: "output_type"
        value: "Shellcode"
      filename: "apollo.bin"
      c2_profiles:
      - c2_profile: "websocket"
        c2_profile_parameters:
          AESPSK: "aes256_hmac"
          callback_host: "ws://192.168.0.118"
          tasking_type: "Push"
      commands:
      - shell
      - exit
      - load
    outputs:
      PayloadUUID: "uuid"
      Whatever: "my thing"
```

In the above example, we have an action of `payload_create`, which means we have a few output keywords that can be replaced. In our `outputs` dictionary, we have `PayloadUUID` that should have the value of `uuid` (this is one of our keywords specifically for `payload_create`), so this value is swapped out with the UUID of the payload we created. We have another output, `Whatever`, that has a value of `my thing` (this isn't one of our keywords, so it's left as is). This means another step that comes after this one can do something like this:

```yaml
    inputs:
      WrapperPayloadUUID: "apollo bin.PayloadUUID"
      APIToken: "mythic.apitoken"
```

Notice we have an input into a step called `WrapperPayloadUUID` that needs the value of the `PayloadUUID` output from the step `apollo bin`. We also have a special scenario where we get the `mythic.apitoken` (a special API token that'll only exist for this step).
