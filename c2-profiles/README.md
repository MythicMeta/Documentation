# C2 Profiles

## What is it?

Command and Control (C2) profiles are the way an agent actually communicates with Mythic to get tasking and post responses. There are two main pieces for every C2 profile:&#x20;

1. Server code - code that runs in a docker container to convert the C2 profile communication specification (twitter, slack, dropbox, websocket, etc) into the corresponding RESTful endpoints that Mythic uses
2. Agent code - the code that runs in a callback to implement the C2 profile on the target machine.

## Where is it?

C2 profiles can be found by going to "Global Configurations" -> "C2 Profiles" from the top navigational bar.&#x20;

## How do they work?

Each C2 profile is in its own docker container, the status of which is indicated by the LED next to the C2 profile's name.&#x20;

Each docker container has a python service running in it that connects to a RabbitMQ message broker to receive tasking. This allows Mythic to modify files, execute programs, and more within other docker containers. Each docker container sends a heartbeat every few seconds to the main Mythic server to indicate it is still up and running. If Mythic fails to get that notification for over 30 seconds, then the led will turn to a flashing red led.

The next few sections will walk through the two main C2 profiles and how they work along with what C2 profile code means for the server and for an agent.

## Where can I find more documentation about them?

The documentation container contains detailed information about the OPSEC considerations, traffic flow, and more for each container when you install the c2 profile. From the C2 Profiles page, you can click the blue document icon to automatically open up the local documentation website to that profile.
