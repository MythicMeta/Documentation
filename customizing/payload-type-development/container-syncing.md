# Container Syncing

## What is it?

When your payload type container starts up, it connects to the rabbitMQ broker system and sends a heartbeat notification to Mythic. Mythic then tries to look up the associated payload type and, if it can find it, will update the last\_checkin time. However, if Mythic cannot find the payload type, then it'll issue a "sync" message to the container. Similarly, when a container starts up, the first thing it does upon successfully connecting to the rabbitMQ broker system is to send its own synced data.

This data is simply a JSON representation of everything about your payload - information about the payload type, all the commands, build parameters, command parameters, browser scripts, etc.

## When does it happen?

Syncing happens at a few different times and there are some situations that can cause cascading sycing messages.

* When a payload container starts, it sends all of its sycned data down to Mythic
* If a container sends a heartbeat and Mythic doesn't recognize the payload type, it'll issue a Sync message back to the container
* If a C2 profile syncs, it'll trigger a re-sync of all Payload Type containers. This is because a payload type container might say it supports a specific C2, but that c2 might not be configured to run or might not have check-ed in yet. So, when it does, this re-sync of all the payload type containers helps make sure that every agent that supports the C2 profile is properly registered.
* When a Payload Type container syncs, it triggers a re-sync of all "Wrapper" payload types. This is because a wrapper might support a payload type that doesn't exist yet in Mythic (configured to not start, hasn't checked in yet, etc). So, when that type does check in, we want to make sure all of the wrapper payload types are aware and can update as necessary.

## Current PayloadType Versions

* 12 (Mythic 2.3.7)
  * `mythic_payloadtype_container==0.1.7`
  * itsafeaturemythic DockerHub Images:
    * itsafeaturemythic/python38\_payload==0.1.1
    * itsafeaturemythic/leviathan\_payload==0.1.1
    * itsafeaturemythic/csharp\_payload==0.1.1
    * itsafeaturemythic/xgolang\_payload==0.1.1
* 11 (Mythic 2.3.6)
  * `mythic_payloadtype_container==0.1.1`
  * itsafeaturemythic DockerHub Images:
    * itsafeaturemythic/python38\_payload==0.0.11
    * itsafeaturemythic/leviathan\_payload==0.0.10
    * itsafeaturemythic/csharp\_payload==0.0.18
    * itsafeaturemythic/xgolang\_payload==0.0.16
* 9 (Mythic 2.2.8)
  * `mythic_payloadtype_container==0.0.45`
  * itsafeaturemythic DockerHub Images:
    * itsafeaturemythic/csharp\_payload==0.0.14
    * itsafeaturemythic/python38\_payload==0.0.7
    * itsafeaturemythic/xgolang\_payload==0.0.12
    * itsafeaturemythic/leviathan\_payload==0.0.7
* 8 ( Mythic 2.2.7)
  * `mythic_payloadtype_container==0.0.44`
  * itsafeaturemythic DockerHub images:
    * itsafeaturemythic/python38\_payload:0.0.6
    * itsafeaturemythic/csharp\_payload:0.0.13
    * itsafeaturemythic/leviathan\_payload:0.0.6
    * itsafeaturemythic/xgolang\_payload:0.0.11
* 7 (Mythic 2.2.6)
  * `mythic_payloadtype_container==0.0.43`
  * itsafeaturemythic DockerHub images:
    * itsafeaturemythic/python38\_payload:0.0.5
    * itsafeaturemythic/csharp\_payload:0.0.12
    * itsafeaturemythic/leviathan\_payload:0.0.5
    * itsafeaturemythic/xgolang\_payload:0.0.10

## Current Translation Container Versions

* 4 (Mythic 2.3.1)
  * `mythic_translator_container==0.0.15`
  * itsafeaturemythic DockerHub Images:
    * itsafeaturemythic/`python38_translator_container`:0.0.4
