---
description: How to use the Scripting API
---

# Scripting

## What is Mythic scripting?

Mythic utilizes a combination of a GoLang Gin webserver and a Hasura GraphQL interface. Most actions that happen with Mythic go through the GraphQL interface (except for file transfers). We can hit the same GraphQL endpoints and listen to the same WebSocket endpoints that the main user interface uses as part of scripting, which means scripting can technically be done in any language.

## Where is it?

Install the PyPi package via pip `pip3 install mythic` . The current mythic package is version `0.1.0rc2`. The code for it is public - [https://github.com/MythicMeta/Mythic\_Scripting](https://github.com/MythicMeta/Mythic\_Scripting)

### How do I know what I can do?

The easiest way to play around with the scripting is to do it graphically - select the hamburger icon (three horizontal lines) in the top left of Mythic, select "Services", then "GraphQL Console". This will open up `/console` in a new tab.&#x20;

From here, you need to authenticate to Hasura - run `sudo ./mythic-cli config get hasura_secret` on the Mythic server and you'll get the randomized Hasura secret to log in. At this point you can browser around the scripting capabilities (API at the top) and even look at all the raw Database data via the "Data" tab.

### Examples

The Jupyter container has a lot of examples of using the Mythic Scripting to do a variety of things. You can access the Jupyter container by clicking on the hamurber icon (three horizontal lines) in the top left of Mythic, select "Services", then "Jupyter Notebooks". This will open up a `/jupyter` in a new tab.

From here, you need to authenticate to Jupyter - run `sudo ./mythic-cli config get jupyter_token` on the Mythic server to get the authentication token. By default, this is `mythic`, but can be changed at any time.
