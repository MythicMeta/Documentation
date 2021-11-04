# Payload Types

## What is it?

Payload types are the different kinds of agents that can be created and used with Mythic.&#x20;

## Where are they located?

Payload type information is located in "Global Configuration" -> "Payload Types" from the top navigation bar.

From this initial high-level view, a few important pieces of information are shown:

* The green light indicates that the corresponding docker container for the payload type has sent a heartbeat within the last 30 seconds. A red light indicates that there has been no heartbeat within the last 30 seconds. There's more information on all of this in the next sections.
* The name of the payload type which must be unique
* Which operating systems the agent supports
* The number of commands associated with the payload type
* The "Commands" section provides the ability to view the commands and their configurations.

{% hint style="info" %}
To modify the Payload Type itself, you need to modify the corresponding class in the Payload Type's docker container. This class will extend the PayloadType class.
{% endhint %}

## Where can I find more documentation about them?

The documentation container contains detailed information about the commands, OPSEC considerations, supported C2 profiles, and more for each payload type when you install it. From the Payload Types page, you can click the blue document icon to automatically open up the local documentation website to that agent.
