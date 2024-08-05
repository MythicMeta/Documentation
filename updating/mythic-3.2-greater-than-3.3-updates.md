---
description: Updates and needed agent changes between versions
---

# Mythic 3.2->3.3 Updates

## Breaking Changes

* All Webhook / Logging containers must be updated to the new PyPi/Go packages and provide two new fields: name and description.
* All C2 Profiles must update to the new PyPi/Go packages so that they can properly handle messages for getting/updating files (including editing the local config). No other change required outside of just updating the packages.
* All Payload Types just need to update to the new PyPi and MythicContainer packages so that they can accept the new file\* RPC calls from the web UI.
  * MythicContainer @ v1.4.0 for Golang
  * mythic-container == 0.5.3 for Python

## Agent Messages to Mythic

**Artifacts**

Two new fields you can (optionally) report back with your artifacts are the following:

* `needs_cleanup` - This identifies in the Mythic UI if this artifact needs some additional action to be cleaned up. Some artifacts naturally clean themselves up or are temporary, others need to be manually removed/killed.
* `resolved` - This indicates that the artifact that needed to be cleaned up has been successfully cleaned up

**File Browser**

* `success` is now an optional boolean field instead of a required one. Setting this to `true` will result in a green checkmark in the UI and setting this to `false` will result in a red warning sign. Not setting a value will leave it with no additional icon.

## Quality of Life

### Invite Links

Admins can generate one-time-use invite links to invite a new operator to their Mythic server without pre-creating the account. This is disabled by default but can be enabled via .env or the global settings by an admin. More info [here](mythic-3.2-greater-than-3.3-updates.md#invite-links).
