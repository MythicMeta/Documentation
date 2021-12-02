# Mythic 2.2 -> 2.3 Updates

If you want to look into all the new features available to you as a payload developer from an agent perspective, check out [agents-2.2-greater-than-2.3.md](agents-2.2-greater-than-2.3.md "mention"). The rest of this page is for higher-level updates and UI changes. Either way, to leverage these updates and to upgrade your Mythic instance, you will need to delete your current database `sudo ./mythic-cli database reset` and then pull in the updates and `sudo ./mythic-cli mythic start` again.

{% hint style="danger" %}
If you're coming from the 2.2.\* Mythic instances, your current agents will NOT work. They will NOT work. This update changed a lot of the underlying ways those agents communicated and synced with mythic, so you'll need to update those. In most cases, once the developer has announced their stuff is updated, you can use the `./mythic-cli install github <url> [branch] [-f]` command to remove the old version and pull in the new version.
{% endhint %}

{% hint style="warning" %}
When updating minor versions (2.2 to 2.3), make sure you drop your database with `sudo ./mythic-cli database reset` and delete your `Mythic/.env` file.
{% endhint %}

## Tasking an Agent

Tasking agents from the web interface has changed slightly. When you start typing, there is no more automatic autocomplete dialog that pops up to help you out. Instead, as you're typing a command (or before you've typed any letters) you can press `Tab` to cycle through available commands. For example, the `apfell` agent has a `shell` and `shell_elevated` command. If you start typing `shel` and hit tab, you'll first get `shell`, then `shell_elevated`, then back to `shell` again.&#x20;

Once you have a command and you type a space, you can start tab-completing the command's parameters. The `apfell` agent's `shell` command takes one parameter, a `String` called `command`. If you type `shell` and hit tab, the web interface will start providing the command parameters for you. In this case, you'd get `shell -command`. At this point, without hitting `space` again, if you continue hitting tab, and if the `shell` command had more parameters, the `Tab` button would cycle through the available parameters until you hit `space` and start typing out the values.

At any time, if a command has parameters, you can hit `shift+enter` and cause the tasking modal to pop up.&#x20;

### Parameter Groups

To provide a form of conditional parameters for commands, Mythic now supports `parameter groups`. This isn't a new concept - Microsoft's PowerShell does a similar thing. For a single command, you define which parameters are "grouped" together. This allows you to say that two parameters can't be used together, or that you can provide parameterA or parameterB depending on if parameterC is provided. The hard part was how to display this sort of meta information to the user.

When you're just typing out parameters on the command line and using the tab complete capabilities described in the previous section, then Mythic will automatically know which parameter group you're using. If you satisfy the requires for multiple parameter groups while using tab-complete, Mythic will just keep giving you options from all matching groups. Once you've supplied enough parameters to match only a single group, then Mythic will only give you parameter recommendations from that group. However, if you're using the popup modal, then Mythic will have a new dropdown menu at the top for selecting which parameter group you want to use. If you've started typing out parameters on the command line and hit `shift+enter` to cause the modal to appear, if Mythic can determine which parameter group you're using, then that one will automatically populate the modal.

## Old / New Web Interface

With the changes to how tasking is working, the old user interface will be decommissioned and you will have to use the new interface. &#x20;
