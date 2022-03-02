# Updating Mythic

## How to update Mythic to a new version

Your current Mythic version is shown in the top right corner when you're logged in.

There are two scenarios to updating Mythic: updates within the minor version (1.4.1 to 1.4.2) and updates to the minor (1.4.1 to 1.5) or major version (1.4 to 2.0).

### Updating within the minor version

This is when you're on version 1.2 for example and want to pull in new updates (but not a new minor version like 1.3 or 1.4). In this case, the database schema should not have changed.

1. Pull in the latest code for your version (if you're still on the current version, this should be as easy as a `git pull`)
2. Restart Mythic to pull in the latest code changes into the docker containers with `sudo ./mythic-cli mythic start`

### Updating minor or major versions

This is when you're on version 1.2 for example and want to upgrade to version 1.3 or 2.1 for example. In this case, the database schema has changed.&#x20;

{% hint style="danger" %}
In order to upgrade, you'll end up losing all of the information in your database, so make sure you download any files, export any reports, or save any tasking you've done. This is irreversible! The entire database will be cleared and reset back to default.
{% endhint %}

1. Reset the database with `sudo ./mythic-cli database reset`
2. Pull in the version you want to upgrade to (if you're wanting to upgrade to the latest, it's as easy as `git pull`)
3. Delete your `Mythic/.env` file - this file contains all of the per-install generated environment variables. There might be new environment variables leveraged by the updated Mythic, so be sure to delete this file and a new one will be automatically generated for you.
4. Restart Mythic to pull in the latest code changes into the docker containers with `sudo ./mythic-cli mythic start`

Since Mythic now has all of the C2 Profiles and Payload Types split out into different GitHub Organizations ([https://github.com/MythicAgents](https://github.com/MythicAgents) and [https://github.com/MythicC2Profiles](https://github.com/MythicC2Profiles)), you might need to update those projects as well.

## Updating Agent or C2 Profile services

Agents and C2 Profiles are hosted in their own repositories, and as such, might have a different update schedule than the main Mythic repo itself. So, you might run into a scenario where you update Mythic, but now the current Agent/C2Profiles services are no longer supported.

You'll know if they're no longer supported because when the services check in, they'll report their current version number. Mythic has a range of supported version numbers for Agents, C2 Profiles, Translation services, and even scripting. If something checks in that isn't in the supported range, you'll get a warning notification in the UI about it.

To update these (assuming that the owner/maintainer of that Agent/C2 profile has already done the updates), simply stop the services (`sudo ./mythic-cli payload stop agentname` or `sudo ./mythic-cli c2 stop profileName`) and run the install command again. The install command should automatically determine that a previous version exists, remove it, and copy in the new components. Then you just need to either start those individual services, or restart mythic overall.
