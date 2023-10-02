# Updating Mythic

## How to update Mythic to a new version

In the Mythic UI, you can click the hamburger icon (three horizontal lines) in the top left to see the current Server version and UI version.&#x20;

There are three scenarios to updating Mythic: updates to the patch version (1.4.1 to 1.4.2), updates to the minor (1.4.1 to 1.5), or major version (1.4 to 2.0).

### Updating patches

This is when you're on version 1.2 for example and want to pull in new updates (but not a new minor version like 1.3 or 1.4). In this case, the database schema should not have changed.

1. Pull in the latest code for your version (if you're still on the current version, this should be as easy as a `git pull`)
2. Make a new mythic-cli binary with `sudo make`
3. Restart Mythic to pull in the latest code changes into the docker containers with `sudo ./mythic-cli start`

### Updating minor or major versions

This is when you're on version 1.2 for example and want to upgrade to version 1.3 or 2.1 for example. In this case, the database schema has changed.&#x20;

Starting with Mythic 3.1, we now have database migrations within PostgreSQL. You should be fine to git pull and rebuild everything. It's important that you rebuild so that server changes are pulled in for the various services that updated. This means that if you're on Mythic 3.0.0-rc\* and want to upgrade to Mythic 3.1.0, you'll automatically get database migrations to help with this.

**Note:** I always highly recommend backing everything up if you plan to update a production system. Just in case something happens, you'll be able to revert.&#x20;

You will have some down time while this happens (the containers need to rebuild and start back up), so make sure whatever you're doing can handle a few seconds to a few minutes of down time.

If you want to wipe the database and upgrade, the following steps will help:

1. Reset the database with `sudo ./mythic-cli database reset`
2. Make sure Mythic is stopped, `sudo ./mythic-cli stop`
3. Purge all of your containers, `sudo docker system prune -a`
4. Pull in the version you want to upgrade to (if you're wanting to upgrade to the latest, it's as easy as `git pull`)
5. Delete your `Mythic/.env` file - this file contains all of the per-install generated environment variables. There might be new environment variables leveraged by the updated Mythic, so be sure to delete this file and a new one will be automatically generated for you.
6. Restart Mythic to pull in the latest code changes into the docker containers with `sudo ./mythic-cli start`

Since Mythic now has all of the C2 Profiles and Payload Types split out into different GitHub Organizations ([https://github.com/MythicAgents](https://github.com/MythicAgents) and [https://github.com/MythicC2Profiles](https://github.com/MythicC2Profiles)), you might need to update those projects as well.

## Updating Agent or C2 Profile services

Agents and C2 Profiles are hosted in their own repositories, and as such, might have a different update schedule than the main Mythic repo itself. So, you might run into a scenario where you update Mythic, but now the current Agent/C2Profiles services are no longer supported.

You'll know if they're no longer supported because when the services check in, they'll report their current version number. Mythic has a range of supported version numbers for Agents, C2 Profiles, Translation services, and even scripting. If something checks in that isn't in the supported range, you'll get a warning notification in the UI about it.

To update these (assuming that the owner/maintainer of that Agent/C2 profile has already done the updates), simply stop the services (`sudo ./mythic-cli stop agentname` or `sudo ./mythic-cli stop profileName`) and run the install command again. The install command should automatically determine that a previous version exists, remove it, and copy in the new components. Then you just need to either start those individual services, or restart mythic overall.
