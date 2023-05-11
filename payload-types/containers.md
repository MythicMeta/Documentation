# Containers

## Where are they?

All installed docker containers are located at `Mythic/InstalledServices/`each with their own folder. The currently running ones can be checked with the `sudo ./mythic-cli status` . Check [A note about containers](../installation/a-note-about-containers.md) for more information about them.

## Why use containers?

Containers allow Mythic to have each Payload Type establish its own operating environment for payload creation without causing conflicting or unnecessary requirements on the host system.

## When do containers come into play?

Payload Type containers only come into play for a few special scenarios:

* Payload Creation
* Tasking
* Processing Responses

For more information on editing or creating new containers for payload types, see [Payload Type Development](../customizing/payload-type-development/).
