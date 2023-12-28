# Mythic 2.3 -> 3.0 Updates

If you want to look into all the new features available to you as a payload developer from an agent perspective, check out [Agents 2.3 -> 3.0](agents-2.3-greater-than-3.0.md).  The rest of this page is for higher-level updates and UI changes.

## Mythic Server

The main goal of this update was to re-do the entire back-end for Mythic from Python3 to GoLang. Mythic's Python back-end was written during a time when asyncio was starting to gain popularity and as such had a mix of naturally asyncio functionality and some faked asyncio functionality. This combination resulted in a database connectivity bug that would only sometimes show up in unreproducible ways and would make it appear as though all of the containers were offline, despite them all being online.

Since the work to change all of the database connectivity code would have touched almost every single function in conjunction with the core of how Mythic operated, it was the perfect time to just redo the back-end and solve a lot of technical debt as well.

## Automatic Deployments

What if you're automatically deploying Mythic 2.3 right now and want to know what changes need to happen on your end to automatically deploy Mythic 3.0? No worries, here are the deployment changes:

* `mythic-cli` isn't included as part of the repo anymore (2.5MB changing binary is gross). Instead, you need to run `sudo make` in the Mythic folder first (that'll create a docker container to build it and then copy it out to the Mythic folder for you automatically).
* Mythic no longer does logging/webhooks directly. Instead, if you want logging and webhooks, you need to install those from the [C2Profiles GitHub](https://github.com/MythicC2Profiles).&#x20;
  * A basic webhook container is: [basic\_webhook](https://github.com/MythicC2Profiles/basic\_webhook)
  * A basic logger container is: [basic\_logger](https://github.com/MythicC2Profiles/basic\_logger)
* If you're scripting anything with Mythic, make sure to update to the latest `mythic` pypi package and double check your scripts still work.

## Agent Development / Containers

Mythic v2.3 used PyPi packages (`mythic_payloadtype_container`, `mythic_c2_container`, and `mythic_translation_container`) and very specific folder layouts to configure and run the various agents, c2 profiles, and translation containers. This became tedious to maintain and resulted in a lot of duplicated effort as the capabilities offered to containers expanded. To address this, now Mythic containers all use `mythic_container` as their PyPi package if they're written in Python or `github.com/MythicMeta/MythicContainer` package if they're written in GoLang.

This new format also allows a single "container" to have multiple payload types, multiple c2 profiles, translation containers, and even two new kinds of containers.&#x20;

### Logging

In an effort to embrace the microarchitecture more, Mythic split out the logging from the main server. Instead, when there is a logging-based event, Mythic will emit that message to a logging container (if one is installed). This allows operators/developers to configure logging to happen however they want. You can write to stdout, to certain files, adjust the format of the messages, you can even use the MythicRPC functionality to enrich the data and log even more. You can even ship the data directly to your SIEM. An example of this is available at `https://github.com/MythicC2Profiles/basic_logger`.

### Webhooks

Continuing the effort of the microarchitecture, Mythic split out webhooks. Right now there are three webhook-able events - new callbacks, mythic starting up, and "feedback". You can configure global settings for these in the Mythic UI with a single webhook URL and channel, but you can also configure the container with per-webhook type custom webhook URLs, channels, and the message itself. Similar to the logging, the webhook simply listens on a RabbitMQ queue for messages and then does something with them. What happens with them is entirely under your control, so you can adjust everything and post to entirely different services if you want. An example of the GoLang version of this logger is available at `https://github.com/MythicC2Profiles/basic_webhook`.

The number and kinds of webhook events will expand over time.

## Agent Features

The rewrite also included a few new agent features.

### Crypto

There's a decision that can be made about where "crypto" lives for an agent. Is the crypto part of the agent core, or is it specific to the c2 profile? Historically, Mythic only supported configuring crypto as part of a c2 profile. Now though, you can mark `crypto_type=true` with a Payload Type's build parameters and supercede a c2 profile's cryptography. This gives the agent developer a higher degree of customizability than existed before.

### Build Parameters

A few new types of parameters were added for building payloads - Dictionaries, Arrays, Numbers, and Dates. Now, any type of parameter that exists for a C2 profile parameter also exists for a build parameter.

### Build Steps

One of the tough things about agent development and usage is that the build process could potentially take a while. However, from the UI, the operator simply sees a spinning circle and has no indication of what's going on - did the container crash? is it still building? how far along in the build process is it?&#x20;

To help with this, Payload Types can now declare "build steps" as part of their Payload Type definition and during the build process they can report back to Mythic that a step is done either successfully or with error and provide both stdout and stderr to go along with it. This makes it a lot easier to see where in the build process you're hitting an error.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 8.52.26 AM.png" alt=""><figcaption></figcaption></figure>

Each one of those steps are clickable and provide detailed information about what's going on along with how long each step took. A Payload's detailed information also shows more detail about the step itself, like the description of what's going on.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 8.54.11 AM.png" alt=""><figcaption></figcaption></figure>

## Services

Mythic now provides an additional services - Jupyter Notebooks. From the hamburger icon in the top left, if you click on "Services", you'll see the services available to you.

### Jupyter

Jupyter Notebooks, `/jupyter`, provide a handy, persistent way to allow scripting without requiring operators to get set up on their own environment. Right now it's a single, shared instance for all of Mythic (not per operation), but if it gains traction an people really like it, there's a beefier version that allows multi-user sign-ins that can be used instead.&#x20;

This will slowly get a library of common scripting examples that you can draw from when creating your own scripts for your operations. This also provides a much nicer interface for testing and scripting than pulling code from a wiki page and hoping it's updated.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 9.07.08 AM.png" alt=""><figcaption></figcaption></figure>

The `password` is `mythic`.

### GraphQL Console

This will open a new tab to `/console` where you can interact with Hasura - the provider for the GraphQL component of Mythic. All of the web UI and scripting goes through Hasura, so this provides a great way to test out custom GraphQL queries and see what all you can do.&#x20;

When you go here you'll be prompted for a credential to log in - use `sudo ./mythic-cli config get hasura_secret` to get the password to use.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 10.56.32 AM.png" alt=""><figcaption></figcaption></figure>

If you generate an API token for yourself via your settings page in the Mythic UI, you can supply that token as shown above so you can see _exactly_ what your account is able to do as if you logged in via scripting.

### Consuming Services

This provides a single page to look at all of the possible logging and webhook message types that can be emitted and allows you to send test messages. This is particularly handy if you're writing/modifying a logging/webhook container and want to make sure that your messages are going through properly.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 9.08.53 AM.png" alt=""><figcaption></figcaption></figure>

## Feedback

As much as possible, operators should be operating. It's very annoying and potentially time consuming if you have to stop what you're doing and record somewhere that you ran into an issue, that you got caught, or even just that something is weird with a tool you're using. To try to help with this, Mythic now has a "feedback" button at the top of the screen (the thumbs down icon).

You can click this at any time and submit to a webhook information for a bug you encountered, record a deconfliction event, submit feedback about something confusing, or even recording a feature request. You must have a webhook container running, like the one from `https://github.com/MythicC2Profiles/basic_webhook` and configure a webhook for your operation, but then it's super easy to record this sort of stuff and continue on with your operation. Then, when your op is done, you can go back and file github issues, feature requests, ask for UI tweaks about things that were confusing, or even having an easy timeline of when you got caught. Because this data is sent to the webhook containers, you can take the data and do whatever you want with it. You don't technically have to do a webhook - you could turn it directly into a github issue or just save it off as a note somewhere for you to visit later.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 10.19.02 AM.png" alt=""><figcaption></figcaption></figure>

## Tagging

Tagging tasks with additional information has been around for a while in Mythic, but it got a bigger facelift this time around. By clicking the tag icon at the top of the screen, you can view all of your current tag types for the operation.&#x20;

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 10.21.14 AM.png" alt=""><figcaption></figcaption></figure>

Tag types consist of a description, a short-hand display, and a color. The color you set will show both the light mode and dark mode so that you can be sure to pick a color that'll show properly regardless of what users prefer.

Once you have tag types created, you can use them in your operation to tag various things with more information. This is most useful in combination with scripting to "auto tag" things, but you can manually tag everything as well. Right now it just helps you see things as your're scrolling through the UI, but soon there will be a dashboard that gives overviews and more detailed information about all of the things that were tagged.

For Tasks, Files, Credentials, Keylogs, Processes, and the FileBrowser, you can click on the tag icon to edit/add tags. This allows you to provide more context to the generic tag type. For example - you can have a tag type of `cred` and if you find credentials in a file, you can then tag that file with the `cred` tag so it's easier for other people to know (and you to remember) that you pulled creds from that file.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 10.52.17 AM.png" alt=""><figcaption></figcaption></figure>

When you create a new tag you select the type of tag, supply your own source of where the data is coming from (some operator, some automated script, etc). You can supply an external-linkable URL for more information or maybe even the target of where the data applies. The JSON data doesn't technically have to be valid JSON, but if it is, then when you click to view the tag it'll be automatically parsed into a nice table.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 10.52.33 AM.png" alt=""><figcaption></figcaption></figure>

you can see the short-hand tag displayed next to the file to let you know that it's been tagged. When you click on the tag `cred` now instead of the tag icon itself, you see the data that was supplied.

<figure><img src="../../.gitbook/assets/Screenshot 2023-03-07 at 10.52.42 AM.png" alt=""><figcaption></figcaption></figure>

## Adjusting loaded commands

Sometimes, especially during development, you want to test out a new feature or expose a new function within Mythic for a payload type. Historically, that meant you either needed to already have a `load` command created, or you'd have to create an entirely new payload, execute it, then test out your new function. That can be a headache and impractical, especially if it's a `script_only` command that you want to make available to your callbacks _now_.&#x20;

To facilitate this, Mythic now allows you to manually adjust which commands are available in your payloads and callbacks through the UI.

{% hint style="warning" %}
This does NOT actually adjust anything within the payload itself, nor does it adjust anything within a running callback. This simply adjusts Mythic's perception of which commands are available. As such, if you use this to add commandX, but it's not actually part of your payload or callback, then your agent won't know what to do with the information. In that case, you'd still need a proper load command to send the new command down to your agent.
{% endhint %}

For payloads, you can select the blue "info" icon next to your payload and scroll down to the "commands" area. You'll see a new button called "Add/Remove Commands":

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Clicking on that will open up a new dialog box where you can select which commands to add and remove:

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

The same flow is available for Callbacks - click the blue down arrow next to an active callback and select the "View Metadata" entry, then scroll down to the loaded commands.
