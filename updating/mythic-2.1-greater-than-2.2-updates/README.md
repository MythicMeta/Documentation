# Mythic 2.1 -> 2.2 Updates

If you want to look into all the new features available to you as a payload developer from an agent perspective, check out [Agents 2.1.\* -> 2.2.2](agents-2.1.-greater-than-2.2.2). The rest of this page is for higher-level updates and UI changes. Either way, to leverage these updates and to upgrade your Mythic instance, you will need to delete your current database `sudo ./mythic-cli database reset` and then pull in the updates and `sudo ./mythic-cli mythic start` again.

{% hint style="danger" %}
If you're coming from the 2.1.\* Mythic instances, your current agents and c2 profiles will NOT work. They will NOT work. This update changed a lot of the underlying ways those agents/c2 profiles communicated and synced with mythic, so you'll need to update those. In most cases, once the developer has announced their stuff is updated, you can use the `./mythic-cli install github <url> [branch] [-f]` command to remove the old version and pull in the new version.
{% endhint %}

{% hint style="warning" %}
When updating minor versions (2.1 to 2.2), make sure you drop your database with `sudo ./mythic-cli database reset` and delete your `Mythic/.env` file.
{% endhint %}

## Split out C2 / PayloadTypes

One of the issues we've had with Mythic in the past is that people just install it and start it, without any additional configuration. There's no issue with this, but this does mean that all payload types and all c2 profiles have their containers pulled down and started. If you're not expecting it, this can be a lot of space. There are ways to configure this, but there needs to be a way thats a bit more aparent. So, to help with this (and to help with versioning/updates overall), all Payload Types and C2 Profiles are now split out from the main Mythic repository.

* Agents - [https://github.com/MythicAgents](https://github.com/MythicAgents)
* C2 Profiles - [https://github.com/MythicC2Profiles](https://github.com/MythicC2Profiles)

To install an agent and c2 profile, you can use the `./mythic-cli install` script and point it to the address for an agent or c2 profile:

```
sudo ./mythic-cli install github https://github.com/MythicAgents/apfell
```

The hope is that by making this an explicit step will allow people to be more cognizent about which agents/c2 profiles they're installing, how much space they're using, and allow payload types/c2 profiles to be updated at a higher frequency than the Mythic server itself.

The format required for these external agents/c2 profiles is documented and templated out for you here: [https://github.com/its-a-feature/Mythic\_External\_Agent](https://github.com/its-a-feature/Mythic\_External\_Agent). Simply mirror that folder structure and config file, then host your agent on github, and everybody will be able to install and leverage your code.

If you're interested in creating new agents or c2 profiles for people to use, let me know on twitter, and I can add you to the appropriate organization so that you have full control over that project. The nice thing about having all of the agents/profiles aggregated under the same organization is that it makes it easier for people to find rather than a bunch of random github links.

## MythicMeta Organization

In addition to moving all of the Payload Types and C2 Profiles into their own Mythic-based organizations on GitHub, a lot of the "meta" information for Mythic that was stored with it is now in its own Mythic organized called [MythicMeta](https://github.com/MythicMeta).

Inside this organization you'll find the code for:

* [Docker Templates](https://github.com/MythicMeta/Mythic\_Docker\_Templates) - This contains the code and files used to create the Docker Images that are used with the standard Mythic Agents. If you're curious what actually makes a `itsafeaturemythic/csharp_payoad:0.0.11` Docker image, this is where you'll find that.
* [Payload Type PyPi Container](https://github.com/MythicMeta/Mythic\_PayloadType\_Container) - This is the code that's in the `mythic_payloadtype_container` PyPi package hosted on PyPi.
* [C2 Profile PyPi Container](https://github.com/MythicMeta/Mythic\_C2\_Container) - This is the code that's in the `mythic_c2_container` PyPi package hosted on PyPi.
* [Translator PyPi Container](https://github.com/MythicMeta/Mythic\_Translator\_Container) - This is the code that's in the `mythic_translator_container` PyPi package hosted on PyPi.
* [Mythic Scripting](https://github.com/MythicMeta/Mythic\_Scripting) - This is the code that's used for the `mythic` (and right now specifically the `mythic_rest`) scripting capabilities and hosted on PyPi.

For any of these, if you don't want to leverage the standard Docker images, or if you want to turn your own VM into a supported container, feel free to leverage this code.

## Mythic-cli

Mythic used to leverage a large number of bash scripts to accomplish the task of start/stoping docker-compose and the agent/c2 profiles, resetting the database, processing various configuration files, and more. While bash is on all Linux systems where Mythic can run, that doesn't mean that all of the additional support binaries exist (jq, realpath, openssl, docker, etc). This can result in a bit of a headache; plus, maintaining bash scripts is a nightmare. To get around this, Mythic now comes with a pre-compiled Golang binary, `mythic-cli`, with the source code available to all at the [Mythic\_CLI](https://github.com/MythicMeta/Mythic\_CLI) repository.

All of the documentation on this website should already be updated to show how to use the `mythic-cli` binary instead of the support scripts, but if you find a place that doesn't, be sure to report it.

As part of this update, Mythic now leverages a single docker-compose file for all containers. Mythic used to leverage docker-compose for all of the core services and then regular `docker run` commands for the agents and c2 profiles. This worked initially, but now that more and more environment variables are randomized and configurable, there needs to be a way to centrally configure everything. So, starting with Mythic 2.2.4, all configuration happens within the `Mythic/.env` file and all containers are in the `Mythic/docker-compose.yml` file. When you install agents, Mythic will automatically parse and update this docker-compose file to add in the necessary information. Similarly, you can add/remove agents/c2 profiles at any time from this file via `mythic-cli {payload|c2} {add|remove} [name]`. If you already have agents installed that you want to register, the `add` command will allow you to update your docker-compose file without having to re-install your agent or c2 profile. You can also use the `mythic-cli {payload|c2} list` feature to show what containers exist within your docker-compose file and which ones exist on disk.

This will take a little bit of time to get used to, but it will be easier for maintenance and expansion going forward than a bunch of bash scripts.

## C2 Profile Updates

C2 profiles gained a new function, `opsec`, where they can take in all of the parameters an operator supplies when creating a payload and determine if they're safe or not. This is an optional function with more detail on the [OPSEC](../../customizing/c2-related-development/server-side-coding/opsec-checks-for-c2-profiles.md) page for C2 Profiles and general [overview](../../operational-pieces/c2-profiles/opsec-checks.md).

## Payload Updates

Payloads in Mythic got a few updates as well to make it easier for developers and analysis.

### deleted payloads

On the Payloads page, if you delete a payload, then that payload can no longer be used to generate callbacks. If that payload tries to callback to Mythic, you'll get a warning in the UI and in the event feed letting you know that a deleted payload is trying to check in. This is helpful for if a payload gets "burned" and you want to make sure it can't flood your system with callbacks.

### rebuild payloads

Once you've created a payload, on the Payloads page there is a new element in the actions dropdown for "trigger a new build". This will take that payload and task Mythic with generating it again. This is helpful when doing development so that you can quickly troubleshoot build errors without having to apply all of your same build settings each time.

### export/import payload builds

If you want to take a specific build configuration and use it across Mythic installs or us it after you've done a `sudo ./mythic-cli database reset`, you can go to the Payloads page and select to export a payload configuration. This takes all the information as needed to generate a payload and saves it as a new JSON file through your browser. Then, at some time later, on the Payloads creation page, there's a new button on the top right to import a configuration. You can load in this JSON file and automatically trigger a new build of that payload.

### ephemeral payloads

Sometimes it's helpful to see payload information for payloads that you didn't build manually. This can be for commands that might build new instances of payloads for spawning, lateral movement, or privilege escalation. These are "ephemeral" payloads within Mythic and are typically hidden from view. On the Payloads page, you can click a button to view ephemeral payloads so that you can see configurations, build stats, etc for these as well. Similarly, if you delete these then they can't be used to generate new callbacks.

### build\_stdout, build\_stderr

Payloads can now track their stdout and stderr in addition to a build\_message. There are a lot of things that go into building dynamic payloads, so it's helpful to track stdout and stderr for later analysis or troubleshooting. This information can be seen from the Payloads page. This can be set during the `build` function as:

```python
result = BuildResponse()
result.build_stderr = "my stderr message"
result.build_stdout = "my stdout message"
result.build_message = "my normal message to the user"
```

## Operator Updates

### password length

All accounts within Mythic now must have a password that's at least 12 characters long. If you try to set a password that's shorter, then you'll get a warning message and the password setting will fail.

### Lockout & Unknown Users

When a user ties to log in with a name that Mythic doesn't know, all operations will get a warning about it. If a user tries to log in 10 times unsuccessfully, their account will be locked and an admin account will need to re-enable it via the Settings tab in the top right. The initial admin account though can't be locked out (how else would anybody ever get in) - instead, this account goes into a throttle phase where you can only do one password attempt a minute. In both cases (throttle and lock out), all operations will get a notification that this is happening.

### Operator creation

We removed the ability for operators to self-create an account via a "register" button and instead require an admin to pre-create accounts (or use scripting to do so) for all users. To do this via scripting, simply:

```python
async def scripting():
    # sample login
    mythic = mythic_rest.Mythic(
        username="mythic_admin",
        password="mythic_password",
        server_ip="192.168.53.128",
        server_port="7443",
        ssl=True,
        global_timeout=-1,
    )
    print("[+] Logging into Mythic")
    await mythic.login()
    operator1 = await mythic.create_operator(mythic_rest.Operator(username="bob", password="mythic_password"))
```

## Architecture Changes

### Multiple Workers

One of the big issues we found for Mythic over time has been that with a large number of callbacks (200+) or with a medium number with a low sleep (like 15 agents at sleep 0), then the performance for Mythic (server and the UI) is pretty noticeably deteriorated. After digging into it, it turns out that Mythic was only ever using a single core due to how the event loop was leveraged. Mythic now spawns multiple worker processes and shares the load across the available CPUs.

This presents a different set of problems from what we were doing before - for example, you can't cache requests or data in memory because that's not shared amongst the worker processes. So, to help with this, we added a small Redis database. This might seem weird that we now have two databases within Mythic - Redis and Postgres. The use cases and data stored on them is wildly different though.

### redis

The Redis database is cleared each time Mythic starts and is used to hold temporary data that's needed by all of the different worker processes. Currently, this is two things:

* JWT refresh tokens
* SOCKS messages

Access to this data needs to be super fast, but doesn't need to be persistent, which is why it's not stored in the Postgres database.

### socks

There have been issues with the current SOCKS implementation within Mythic, so with the help of Thiago Mallart's pull request for Reverse Port Forwarding, updated the implementation within Mythic to not leverage an external binary, but instead do it all with threads in the main Mythic server. Once we put this implementation through more intensive testing, we will be pulling in Thiago's pull request for reverse port forwarding as well.

## Translation Containers

One of the things Mythic strives to do is allowing an extensible and customizable framework for you to create an agent that functions however you want. While the current Mythic format allows you to generate an agent however you want, the messages that you use are still pretty heavily tied to Mythic's JSON format. Depending on your agent and language of choice, JSON might not be feasible. Even if JSON is feasible, you might not want to use Mythic's JSON messages. So, to help make it easier for your agents to do their own thing, we're introducing the idea of "Translation" containers.

These containers simply act as a way to "translate" between your custom format and the JSON that Mythic needs. In addition to just doing a translation of messages, this container can also handle encryption, decryption, and generation of crypto keys.

If you issue commands to your agent and don't want to adhere to Mythic's format for hooking into features (like the web browser), but do want to utilize that feature, you can use the `process_response` key in your post\_response messages ([Process Response](../../customizing/payload-type-development/process-response.md)) and have your own custom messages sent back to your Command's Python file and then use Mythic's RPC functionality to register the same information. This really does give you the ability to do pretty much everything custom, while still hooking into Mythic.

Translation containers can sit side-saddle with your Payload Type locally in Mythic as well as in your Github repository so that it's easy to bundle and install them together.

More information about translation containers can be found on the [Translation Containers](../../customizing/payload-type-development/translation-containers.md) page.

## Event Feed

The event feed for Mythic got a bunch of updates to help with some of the issues people have faced recently.

### limit fetches

There have been instances where people have tens of thousands of messages in the event feed, which was causing page loads to be really slow. So, Mythic now only fetches the latest 100 messages and has a button at the top to fetch the previous 100. This allows you to scroll back without having to load it all in at once.

### jump to next error

Since all of the messages aren't loaded into the browser at once, it can be hard to find a warning message from Mythic. So, in addition to a button to fetch the previous 100 messages, there's a button to fetch the next most recent error. This makes it easy to find the unresolved errors while still making things much more performant in the browser.

### debug events

When developing, it's often very helpful to see messages as they travel through Mythic. There are a lot of things going on between getting messages, decrypting, potentially sending to translation containers, processes requests, bundling it all back up, etc. If you pass in an environment variable of `MYTHIC_DEBUG=True` when starting Mythic, then all of these stages will send event messages to the event feed with context and information. This can be extremely helpful during development, but will be very overwhelming during production.

### grouped messages

One of the issues we saw people face is an overwhelming number of alert messages. Specifically, when Mythic is too open to the internet and gets scanned, Mythic will report back that it can't find the agent messages in all of this scanning traffic. As an operator, you should still be notified that something is sending messages to you that Mythic can't process, but we don't want to bog down the system. So, Mythic will now "group" like messages. This only happens for "warning" messages, but instead of creating a new event feed entry and a new popup in the UI, Mythic will now increment a count assocaited with the message. If you mark this warning as "resolved" and you get another one of these messages, then you will get another event entry and another popup.

## Going Forward Updates

There were a lot of updates for this Mythic update, and there were a few things put in place to support some of the updates going forward.

### GraphQL

As Mythic expands and provides more context tracking and features, the back-end database will continue to expand. The current UI relies on a REST-style back-end. While this was easy to set up and useful initially, it means that we have to add a bunch more web routes as we add new features and makes maintaining it / testing it more tedious. If we need data in a different format or a different sub-section of data, we have to create new routes and expose them that way. Instead, if we just had a way to expose one endpoint and have the client describe the data it wants, then we can more easily add features to the UI without having to adjust the main Mythic server as well.

This is where GraphQL comes into play. This takes a little bit to get used to, but Mythic now includes the Hasura docker container to expose parts of the Postgres database via a GraphQL engine. This has its own permission model and relies on the main Mythic server for authentication.

### Nginx

As we continue to add more components to Mythic (Documentation container, Mythic server, GraphQL, etc), it's not practical to keep exposing multiple ports that you then need to lock down separately. Instead, we can open a single port externally and proxy connections back to all the components that need them. To accomplish this, we now include an Nginx docker container as a reverse proxy. You can still reach Mythic via the normal `https://mythic_ip:7443`, but now your connections are transparently proxied back to the documentation container and the graphql container for you.

### React UI

Now that Mythic (formerly Apfell) has been out for a few years, I decided to take a look back at how the UI worked. I learned a lot over the years and decided that it was time to redo the UI. It currently is a mixture of Jinja2, Vue, JavaScript, Jquery, and requires adding new templates/routes to the Mythic Server in order to create new pages. It's not particularly easy to go through and trace data or update for people. So, to help with this (and to leverage the new GraphQL components), we're slowly adding a new UI via React.

Using React makes the UI more extensible and makes it easier for other people to contribute. This is going to be a slow process, so for a while there will be two UIs available to you. If you browse to Mythic like normal, you'll see the old/current UI and would never know that there's a new UI as well. If you browse to `/new/login` then you'll be presented the login for the new UI and can start seeing where Mythic is going in the future. We've already started implementing some of our newer features in this React UI such as:

* better Graph views of callbacks via dagre (built on d3) rather than the current manual D3 force directed graphs
* representing some sleep info in the table view for callbacks
* indicating if callbacks have direct routes to Mythic, if there are linked agents, and if there's still a route at all
