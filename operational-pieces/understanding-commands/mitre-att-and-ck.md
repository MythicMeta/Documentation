# MITRE ATT\&CK in Commands

MITRE ATT\&CK is a great way to track what both offense and defense are doing in the information security realm. To help Mythic operators keep track, each command can be tagged with its corresponding MITRE ATT\&CK information:

There can be as many or as few mappings as desired for each command. This information is used in two different ways, but both located in the MITRE ATT\&CK button at the top.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-06 at 9.51.32 AM.png" alt=""><figcaption></figcaption></figure>

The "Fetch All Commands Mapped to MITRE" button takes this information to populate out what is the realm of `possible` with all of the payload types and commands registered within Mythic. This gives a coverage map of what could be done. Clicking each matrix cell gives a breakdown of which commands from which payload types achieve that objective:

The "Fetch All Issued Tasks Mapped to MITRE" only shows this information for commands that have already been executed in the current operation. This shows what's been done, rather than what's possible. Clicking on a cell with this information loaded gives the exact task and command arguments that occurred with that task:
