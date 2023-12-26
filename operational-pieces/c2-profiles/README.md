# C2 Profiles

## What is it?

Command and Control (C2) profiles are the way an agent actually communicates with Mythic to get tasking and post responses. There are two main pieces for every C2 profile:

1. Server code - code that runs in a docker container to convert the C2 profile communication specification (twitter, slack, dropbox, websocket, etc) into the corresponding RESTful endpoints that Mythic uses
2. Agent code - the code that runs in a callback to implement the C2 profile on the target machine.

## Where is it?

C2 profiles can be found by going to Payload Types and C2 Profiles (headphone icon) from the top navigational bar.

## How do they work?

Each C2 profile is in its own docker container, the status of which is indicated on the C2 Profiles page.

Each docker container has a python or golang service running in it that connects to a RabbitMQ message broker to receive tasking. This allows Mythic to modify files, execute programs, and more within other docker containers.&#x20;

## Where can I find more documentation about them?

The documentation container contains detailed information about the OPSEC considerations, traffic flow, and more for each container when you install the c2 profile. From the C2 Profiles page, you can click the blue document icon to automatically open up the local documentation website to that profile.
