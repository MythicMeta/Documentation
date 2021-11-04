# Basic Information

To view a command of a particular payload type, access it via the following:

* Global Configurations -> Payload Types
* Select the view button for the payload type you're interested in
* Use the "Select" option at the top to pick the exact command

Take the following Apfell `upload` command for example:

The `description` provides some context as to what the command does or how it does it. The `help` section shows how to call the command. These two fields are presented the user if they type `help upload` while interacting with a callback. The toggles indicate that this command does not need admin level permissions to execute and that this command does not exit the callback. The command that does have the toggle switched for `is exit` will be tasked to the agent if the user select the GUI option to `exit callback`.&#x20;

The other sections are discussed in greater detail in the following sections.

## Built-in Commands

All PayloadTypes get 3 commands for free - `clear` , `set`. The reason these two commands are 'free' is because they don't actually make it down to the agent itself. Instead, they cause actions to be taken on the Mythic server.

### Clear

The clear command does just that - it clears tasks that are sitting in the queue waiting for an agent to pick them up. It can only get tasks that are in the submitted stage, not ones that are already to the processing stage because that means that an agent has already requested it.

`clear` - entering the command just like this will clear the last task you entered

`clear all` - entering the command just like this will clear `all` tasks you've entered on that callback that are in the appropriate stages.

`clear #` - entering the command just like this will attempt to clear the task indicated by the number after clear.

If a command is successfully cleared by this command before an agent can get to it, then that task will get an automated response stating that it was cleared and which operator cleared it. The clear task itself will get back a list of all the tasks it cleared.

### Set

The set command allows you to set certain properties on a callback:

`set description [some description here]` - this allows you to adjust the description for a callback for all operators

`set description reset` - this will reset the description back to the default `tag` value specified when creating the associated payload
