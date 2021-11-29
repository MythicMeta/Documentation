# Mythic 2.2 -> 2.3 Updates

If you want to look into all the new features available to you as a payload developer from an agent perspective, check out [agents-2.2-greater-than-2.3.md](agents-2.2-greater-than-2.3.md "mention"). The rest of this page is for higher-level updates and UI changes. Either way, to leverage these updates and to upgrade your Mythic instance, you will need to delete your current database `sudo ./mythic-cli database reset` and then pull in the updates and `sudo ./mythic-cli mythic start` again.

{% hint style="danger" %}
If you're coming from the 2.2.\* Mythic instances, your current agents will NOT work. They will NOT work. This update changed a lot of the underlying ways those agents communicated and synced with mythic, so you'll need to update those. In most cases, once the developer has announced their stuff is updated, you can use the `./mythic-cli install github <url> [branch] [-f]` command to remove the old version and pull in the new version.
{% endhint %}

{% hint style="warning" %}
When updating minor versions (2.2 to 2.3), make sure you drop your database with `sudo ./mythic-cli database reset` and delete your `Mythic/.env` file.
{% endhint %}

