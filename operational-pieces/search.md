# Search

## Where is it?

The operational search feature can be found by navigating "Operational Views" -> "Search" from the top navigational bar. Searching can be done across commands an operator typed, the output of commands, or comments on tasking.

## How is it used?

The search bar checks for what you type as a regex expression

![](<../.gitbook/assets/Screen Shot 2020-07-13 at 10.58.09 AM.png>)

The above screenshot shows a search of command output for the word "its-a-feature", but the search bar accepts regex as well. You can search the following ways:

* Responses
  * Searches output from all tasking in your current operation
* Command Parameters&#x20;
  * Searches just tasking parameters for specific regex. This can be further limited to just commands executed by certain operators.&#x20;
* Commands&#x20;
  * Search just the command names. This can be further limited to just commands executed by certain operators. This is useful if you want to see all instances of a specific command being executed across the operation.
* Comments
  * Search just the comments associated with tasks across the current operation.
* File Browser
  * Search the file browser objects that are stored for the current operation
* Event Log
  * Search through the event log messages (even those that have been deleted)
