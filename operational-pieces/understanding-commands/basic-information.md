# Basic Information

## Built-in Commands

All PayloadTypes get 2 commands for free - `clear` and `help`. The reason these two commands are 'free' is because they don't actually make it down to the agent itself. Instead, they cause actions to be taken on the Mythic server.

### Clear

The clear command does just that - it clears tasks that are sitting in the queue waiting for an agent to pick them up. It can only get tasks that are in the submitted stage, not ones that are already to the processing stage because that means that an agent has already requested it.

`clear` - entering the command just like this will clear all of the tasks in that callback are in the appropriate stages.

`clear all` - entering the command just like this will clear `all` tasks you've entered on that callback that are in the appropriate stages.

`clear #` - entering the command just like this will attempt to clear the task indicated by the number after clear.

If a command is successfully cleared by this command before an agent can get to it, then that task will get an automated response stating that it was cleared and which operator cleared it. The clear task itself will get back a list of all the tasks it cleared.

### **help**

The `help` command allows users to get lists of commands that are currently loaded into the agent. Just `help` gives basic descriptions, but `help [command]` gives users more detailed command information. These commands look at the loaded commands for a callback and looks at the backing Python files for the command to give information about usage, command parameters, and elevation requirements.
