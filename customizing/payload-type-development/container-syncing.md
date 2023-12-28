# Container Syncing

## What is it?

When your container starts up, it connects to the rabbitMQ broker system. Mythic then tries to look up the associated payload type and, if it can find it, will update the running status. However, if Mythic cannot find the payload type, then it'll issue a "sync" message to the container. Similarly, when a container starts up, the first thing it does upon successfully connecting to the rabbitMQ broker system is to send its own synced data.

This data is simply a JSON representation of everything about your payload - information about the payload type, all the commands, build parameters, command parameters, browser scripts, etc.

## When does it happen?

Syncing happens at a few different times and there are some situations that can cause cascading syncing messages.

* When a payload container starts, it sends all of its synced data down to Mythic
* If a C2 profile syncs, it'll trigger a re-sync of all Payload Type containers. This is because a payload type container might say it supports a specific C2, but that c2 might not be configured to run or might not have check-ed in yet. So, when it does, this re-sync of all the payload type containers helps make sure that every agent that supports the C2 profile is properly registered.
* When a Wrapper Payload Type container syncs, it triggers a re-sync of all non-wrapper payload types. This is because a payload type might support a wrapper that doesn't exist yet in Mythic (configured to not start, hasn't checked in yet, etc). So, when that type does check in, we want to make sure all of the wrapper payload types are aware and can update as necessary.

## Current Container Versions

Latest versions can always be found on the Mythic [README](https://github.com/its-a-feature/Mythic).
