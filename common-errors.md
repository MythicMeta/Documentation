---
description: Discussion / Explanation for Common Errors
---

# Common Errors

## Payload container, X, of version Y is not supported

![Payload container version not supported](<.gitbook/assets/Screen Shot 2021-07-08 at 1.10.23 PM.png>)

All Payload containers leverage the `mythic_container` [PyPi package](https://github.com/MythicMeta/Mythic\_PayloadType\_Container) or the `github.com/MythicMeta/MythicContainer` golang package. These packages keeps track of a version that syncs up with Mythic when the container starts. As Mythic gains new functionality or changes how things are done, these containers might not be supported anymore. At any given time, Mythic could support only a single version or a range of versions. A list of all PyPi reported versions and their corresponding Mythic version/DockerImage versions can be found [here](customizing/payload-type-development/container-syncing.md#current-payloadtype-versions).

### How do I fix this?

The agent in question needs to have its container updated or downgraded to be within the range specified by your version of Mythic. If you're using a Dockerimage from `itsafeaturemythic` (i.e. in your `Mythic/Payload_Types/[agent name]/Dockerfile` it says `FROM itsafeaturemythic/something`) then you can look [here](customizing/payload-type-development/container-syncing.md#current-payloadtype-versions) to see which version you should change to.

## C2 Profile's internal server stopped

![http's internal server stopped](<.gitbook/assets/Screen Shot 2021-07-08 at 1.19.29 PM.png>)

This is a warning that a C2 Profile's internal service was started, but has since stopped. Typically this happens as a result of rebooting the Mythic server, but if for some reason a C2 Profile's Docker container restarts, you'll get this notification as well.

If a C2 Profile is manually stopped by an operator instead of it stopping automatically for some other reason, the warning message will reflect the variation:

![mythic\_admin stopped the c2 profile container](<.gitbook/assets/Screen Shot 2021-07-08 at 1.24.21 PM.png>)

### How do I fix this?

Go to the C2 Profiles page and click to "Start Internal Server". If the container went down and is still down, then you won't be able to start it from the UI and you'll see a different button that you "Can't Start Server". If that's the case, you need to track down why that container stopped on the host.

## Failed to correlate UUID, X, to something Mythic knows

![Failed to correlate UUID](<.gitbook/assets/Screen Shot 2021-07-08 at 1.30.29 PM.png>)

The "Failed to correlate UUID" message means that data came in through some C2 Profile, made its way to Mythic, Mythic base64 decoded the data successfully and looked at the first characters for a UUID. In Mythic messages, the only piece that's normally not encrypted is this random UUID4 string in the front that means something to Mythic, but is generally meaningless to everybody else. Mythic uses that UUID to look up the callback/payload/stage crypto keys and other related information for processing. In this case though, the UUID that Mythic sees isn't registered within Mythic's database. Normally people see this because they have old agents still connecting in, but they've since reset their database.

Looking at the rest of the message, we can see additional data. All C2 Profile docker containers add an additional header when forwarding messages to the Mythic server with `mythic: c2profile_name`. So, in this case we see `'mythic': 'http'` which means that the `http` profile is forwarding this message along to the Mythic server.

### How do I fix this?

First check if you have any agents that you forgot about from other engagements, tests, or deployments. The error message should tell you where they're connecting from. If the UUID that Mythic shows isn't actually a UUID format, then that means that some other data made its way to Mythic through a C2 profile. In that case, check your Firewall rules to see if there's something getting through that shouldn't be getting through. This kind of error does not impact the ability for your other agents to work (if they're working successfully), but does naturally take resources away from the Mythic server (especially if you're getting a lot of these).

## Exec user process caused: no such file or directory

If you are going back-and-forth between windows and linux doing edits on files, then you might accidentally end up with mixed line endings in your files. This typically manifests after an edit and when you restart the container, it goes into a reboot loop. The above error can be seen by using `sudo ./mythic-cli logs [agent name]`.&#x20;

### How do I fix this?

Running `dos2unix` on your files will convert the line endings to the standard linux `\n` characters and you should then be able to restart your agent `sudo ./mythic-cli start [agent name]`. At that point everything should come back up.
