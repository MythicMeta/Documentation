---
description: Internal documentation docker container
---

# Internal Documentation

## Granular Documentation

Mythic provides an additional docker container that serves static information about the agents and c2 profiles. In the main `Mythic/.env` file you'll see which port you wan to run this on via the "DOCUMENTATION\_PORT" key. You don't need to worry about this port too much since Mythic uses an Nginx reverse proxy to transparently proxy connections back to this container based on the web requests you make.

## Accessing the Documentation

The documentation stands up a Golang HTTP server based on Hugo (it's HTTP, not HTTPS). It reads all of the markdown files in the `/Mythic/documentation-docker/` folder and creates a static website from it. For development purposes, if you make changes to the markdown files here, the website will automatically update in real time. From the Mythic UI if you hit `/docs/` then you'll be directed automatically to the main documentation home page.
