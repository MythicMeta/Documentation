# MITRE ATT\&CK

## What is it?

MITRE ATT\&CK ([https://attack.mitre.org/](https://attack.mitre.org)) is an amazing knowledge base of adversary techniques.

> MITRE ATT\&CK® is a globally-accessible knowledge base of adversary tactics and techniques based on real-world observations. The ATT\&CK knowledge base is used as a foundation for the development of specific threat models and methodologies in the private sector, in government, and in the cybersecurity product and service community.
>
> With the creation of ATT\&CK, MITRE is fulfilling its mission to solve problems for a safer world — by bringing communities together to develop more effective cybersecurity. ATT\&CK is open and available to any person or organization for use at no charge.

## Where is it?

This is in development to bring into the new user interface. This is still tracked by the back-end and available via reporting, but the ATT\&CK matrix itself still needs to be ported over to the new React interface.

## How does this Task mapping happen?

Commands can be automatically tagged with MITRE ATT\&CK Techniques (this is what populates the "Commands by ATT\&CK" output). To locate this, you just need to look at the associated python/golang files for each command.

In addition to this file defining the general properties of the command (such as parameters, description, help information, etc). There's a field called `attackmapping` that takes an array of MITRE's `T#` values. For example, looking at the `apfell` agent's `download` command:

```python
class DownloadCommand(CommandBase):
    cmd = "download"
    needs_admin = False
    help_cmd = "download {path to remote file}"
    description = "Download a file from the victim machine to the Mythic server in chunks (no need for quotes in the path)."
    version = 1
    author = "@its_a_feature_"
    parameters = []
    attackmapping = ["T1020", "T1030", "T1041"]
    argument_class = DownloadArguments
    browser_script = BrowserScript(script_name="download", author="@its_a_feature_")
```

When this command syncs to the Mythic server, those T numbers are stored and used to populate the ATT\&CK Matrix. When you issue this `download` command, Mythic does a lookup to see if there's any MITRE ATT\&CK associations with the command, and if there are, Mythic creates entries for the "Tasks by ATT\&CK" mappings. This is why you're able to see the exact command associated.

## How do I update this to add/remove mappings?

As long as you're keeping with the _old_ MITRE ATT\&CK mappings, simply add your T# to the list like shown above, then run `sudo ./mythic-cli start [agent name]`. That'll restart the agent's container and trigger a re-sync of information. If the container is using golang instead of python for its Mythic connectivity, then you need to run `sudo ./mythic-cli build [agent name]` instead.
