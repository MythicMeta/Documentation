# MITRE ATT\&CK

## What is it?

[MITRE ATT\&CK](https://attack.mitre.org) is a knowledge base of adversary tactics and techniques mapped out to various threat groups. It provides a common language between red teams and blue teams when discussing operations, TTPs, and threat hunting. For Mythic, this provides a great way to track all of the capabilities the agents provide and to track all of the capabilities so far exercised in an operation.

For more information on MITRE ATT\&CK, check out the following:

* [https://attack.mitre.org](https://attack.mitre.org)
* [https://twitter.com/mitreattack](https://twitter.com/mitreattack)
* [https://attackevals.mitre.org/](https://attackevals.mitre.org)

## Where is it?

[MITRE ATT\&CK](https://attack.mitre.org) integrations can be found at "Reporting" -> "ATT\&CK Mappings" from the top navigation bar.&#x20;

![](<../.gitbook/assets/Screen Shot 2020-08-20 at 11.04.41 AM.png>)

## How to use it?

There are a few different ways to leverage this information within Apfell.

### Commands by ATT\&CK

Clicking the "Commands by ATT\&CK" button will highlight all of the matrix cells that have a command registered to that ATT\&CK technique. Clicking on a specific cell will bring up more specific information on which payload type and which command is mapped to that technique. All of this information comes from the [MITRE ATT\&CK](../understanding-commands/mitre-att-and-ck.md) portion of commands.

![](<../.gitbook/assets/Screen Shot 2020-08-20 at 11.05.18 AM.png>)

### Tasks by ATT\&CK

This is a slightly different view than the "Commands by ATT\&CK" button. This button will highlight and show the cells that have been exercised in the current operation. A cell will only be highlighted if a command was executed in the current operation with that ATT\&CK technique mapped to it. The only other way to get these cells to highlight for this specific view is to do the Task -> ATT\&CK Technique mapping via regex (the next section).

![](<../.gitbook/assets/Screen Shot 2020-08-20 at 11.05.44 AM.png>)

The cell view will show the exact command that caused the cell to be highlighted with a link (via task number) back to the full display of the task:

![Command-Line interface mappings for the current operation in ATT\&CK](<../.gitbook/assets/Screen Shot 2019-07-28 at 6.48.24 PM.png>)

If there is an issue with the mapping, clicking the red X button will remove the mapping.

### Applying Manual Mappings

For some techniques, it's hard to know ahead of time just how to do a full mapping. For example, consider the above screenshot for the ATT\&CK technique of "Command-Line Interface". Technically those are executing commands via the command-line, so the mappings works; however, additional mappings can be made based on the commands be executed. If the command is `whoami` or `id` then it's reasonable to have a secondary ATT\&CK mapping of "System Owner/User Discovery". This is something potentially only known after the fact of execution.

The top portion of the screen uses regex to match tasks in the operation and display them to the user. The regex is applied to both the original parameters and to any modified parameters. This allows the user to search for the `whoami` task without having to be explicit about the base64 obfuscated version if they used it:

![search for "who" regex across tasks](<../.gitbook/assets/Screen Shot 2019-07-28 at 6.59.58 PM.png>)

Once the you've finalized the regex query to only match the desired tasks, select the technique you want to apply from the dropdown, toggle the "finalize" switch, and submit. The new cell should be highlighted when you click "Tasks by ATT\&CK", and you can re-run your search query to confirm what you expected:

![search for "who" regex across tasks updated](<../.gitbook/assets/Screen Shot 2019-07-28 at 7.01.19 PM.png>)

If something went wrong and you want to undo a mapping, simply click "Tasks by ATT\&CK", click the ATT\&CK technique you want to modify, and click the red X button next to any you want to remove.
