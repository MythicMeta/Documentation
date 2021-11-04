# Task Status

## What is Task status

You probably noticed as you used Mythic that there's a status associated with your task. This status goes through a variety of words/colors depending on where things are in the pipeline and what the agent has done with the task. This provides a way for the operator to know what's happening behind the scenes.

## What are the default statuses?

By default, a task goes through the following stages with the following statuses:

1. `preprocessing`  - The task is being sent to the Payload Type container for processing (parsing arguments, confirming values, doing RPC functionality to register files, etc)
2. `submitted` - The task is now ready for an agent to pick it up
3. `processing` - The task has been picked up by an agent, but there hasn't been any response back yet
4. `processed` - The task has at least one response, but the agent hasn't marked it as done yet.
5. `completed` - The task is marked as completed by the agent and there wasn't an error in execution
6. `error` - The agent report back a status of "error"

### Can I set my own status?

The agent can set the status of the task to anything it wants as part of its normal `post_response` information. Similarly, in a task's `create_tasking` function, you're free to set the `task.status` value to one of the following:

* MythicStatus.Success
* MythicStatus.Error
* MythicStatus.Completed
* MythicStatus.Processed
* MythicStatus.Processing

If you set the status to anything other than `MythicStatus.Success` (which is the default), then the task won't be in the `submitted` state and thus won't be picked up by your agent.&#x20;

