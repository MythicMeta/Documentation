# Payload Types

## What is it?

Payload types are the different kinds of agents that can be created and used with Mythic.

## Where are they located?

Payload type information is located in the C2 Profiles and Payload Types page by clicking the headphone icon in the top of the page.

From this initial high-level view, a few important pieces of information are shown:

* Container status indicates if the backing container is online or offline based on certain RabbitMQ Queues existing or not. This status is checked every 5 seconds or so.
* The name of the payload type which must be unique
* Which operating systems the agent supports

{% hint style="info" %}
To modify the Payload Type itself, you need to modify the corresponding class in the Payload Type's docker container. This class will extend the PayloadType class.
{% endhint %}

## Where can I find more documentation about them?

The documentation container contains detailed information about the commands, OPSEC considerations, supported C2 profiles, and more for each payload type when you install it. From the Payload Types page, you can click the blue document icon to automatically open up the local documentation website to that agent.
