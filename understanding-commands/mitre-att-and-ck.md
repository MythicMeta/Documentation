# MITRE ATT\&CK in Commands

MITRE ATT\&CK is a great way to track what both offense and defense are doing in the information security realm. To help Mythic operators keep track, each command can be tagged with its corresponding MITRE ATT\&CK information:

![upload MITRE ATT\&CK mappings](<../.gitbook/assets/Screen Shot 2019-07-22 at 10.42.15 PM.png>)

There can be as many or as few mappings as desired for each command. This information is used in two different ways, but both located in "Reporting" -> "ATT\&CK Mappings" from the top navigation bar.

The "Commands by ATT\&CK" button takes this information to populate out what is the realm of `possible` with all of the payload types and commands registered within Mythic. This gives a coverage map of what could be done. Clicking each matrix cell gives a breakdown of which commands from which payload types achieve that objective:

![T1059 ATT\&CK Mappings](<../.gitbook/assets/Screen Shot 2019-07-22 at 10.45.50 PM.png>)

The "Tasks by ATT\&CK" only shows this information for commands that have already been executed in the current operation. This shows what's been done, rather than what's possible. Clicking on a cell with this information loaded gives the exact task and command arguments that occurred with that task:

![Tasks by ATT\&CK](<../.gitbook/assets/Screen Shot 2019-07-22 at 10.47.40 PM.png>)
