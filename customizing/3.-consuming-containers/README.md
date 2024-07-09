---
description: >-
  Consuming containers are a separate class of containers that wait for things
  to happen that aren't tasking
---

# 3. Consuming Containers

## Location

Consuming containers are located by clicking the hamburger icon in the top left, selecting "services", and then clicking "consuming services".&#x20;

## Types

There are 4 kinds of consuming containers:

* `webhook` - these containers get messages pertaining to alerts, callbacks, feedback, startup, and custom messages. Their goal is to take the data presented to them in these messages and send webhook messages to additional services (like slack/discord)
* `logging` - these containers get messages pertaining to artifacts, callbacks, credentials, files, keylogs, payloads, and tasks. The goal of these containers is to take these messages and log them to files, stdout, or to SIEMs so that these important events can be tracked more easily for your environment
* `eventing` - these containers get messages about custom functions and conditional checks in eventing workflows. They are used to do more complex decisions and actions within workflows than the basic functionality provided by Mythic's core.
* `auth` - these containers extend the login functionality within Mythic. You can either add SSO support or custom auth (such as LDAP), but at the end of the process you have to return the email of the user to authenticate. This email is then checked against the operator's email addresses in Mythic to determine which account to create a JWT for.

## Commonalities

Every consuming container has the following in common:

* `name` - because each of these containers are tracked for their online/offline status (and potentially used for eventing/auth), each one needs to have a unique name
* `description` - it's helpful to describe what each container is responsible for, especially if you have a bunch of services installed to know what's happening where
* `subscriptions` - you'll probably see a "Subscriptions" field, but you don't need to fill this out. Mythic uses this to track what all the container is subscribing to. This is auto populated by the golang/python library code for syncing to Mythic
