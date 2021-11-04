# Linking Agents

## What does it mean to link agents

This refers to the act of connecting two agents together with a peer-to-peer protocol. The details here are agnostic of the actual implementation of the protocol (could be SSH, TCP, Named Pipes, etc), but the goal is to provide a way to give one agent the context it needs to `link` or establish some peer-to-peer connectivity to a running agent.&#x20;

{% hint style="info" %}
This also comes into play when trying to connect to a new executed payload that hasn't gone through the checkin process with Mythic yet to get registered as a Callback.&#x20;
{% endhint %}

### Getting the Linking information

When creating a command, give a parameter of type `ParameterType.ConnectionInfo`. Now, when you type your command without parameters, you'll get a popup like normal. However, for this type, there will be three dropdown menus for you to fill out:

![](<../../.gitbook/assets/Screen Shot 2020-12-10 at 11.54.14 AM.png>)

#### Host:

This field is auto populated based on two properties:

* The list of all hosts where you have registered callbacks
* The list of all hosts where Mythic is aware you've moved moved a payload to

#### Payload:

Once you've selected a host, the `Payload` dropdown will populate with the associated payloads that Mythic knows are on that host. These payloads are in two main groups:

* The payloads that spawned the current callbacks on that host
* The payloads that were moved over via a command that created a new payload and registered it to the new host

This payload simply acts as a template of information so that you can select the final piece.

#### What if your host/payload isn't listed?

Select the green `+` next to host, manually specify which host your payload lives on, then select from the dropdown the associated payload that was used. Then click add. Now Mythic is also tracking that the selected payload lives on the indicated host. You can continue with the host/payload/c2\_profile dropdowns like normal.

![](<../../.gitbook/assets/Screen Shot 2020-12-10 at 11.54.59 AM.png>)

#### C2 Profile:

When trying to connect to a new agent, you have to specify which specific profile you're wanting to connect to. This is because on any given host and for any given payload, there might be multiple c2 profiles within it (think HTTP, SMB, TCP, etc). This field will auto populate based on the C2 profiles that are in the payload selected in the drop down above it.

{% hint style="info" %}
You'll only be able to select C2 profiles that are marked as `is_p2p` for peer-to-peer profiles. This is because it doesn't make any sense to remotely link to an HTTP callback profile for example.&#x20;
{% endhint %}

### Submitting the task:

Once you've selected all of the above pieces, the task will insert all of that selected profile's specific instantiations as part of the task for the agent to use when connecting. This can include things like specific ports to connect to, specific pipe names to use, or any other information that might be needed to make the connection.

### Shorthand:

All of the above is to help an operator identify exactly which payload/callback they're trying to connect to and via which p2p protocol. As a developer, you have the freedom to instead allow operators to specify more generic information via the command-line such as: `link-command-name hostname` or `link-command-name hostname other-identifier`. The caveat is this now requires the operator to know more detailed information about the connection ahead of time.&#x20;

## Leveraging Current/Old Links

The `ParameterType.ConnectionInfo` parameter type is useful when you want to make a new connection between a callback to a payload you just executed or to another callback that your current callback hasn't connected to before. A common command that leverages this parameter type would be `link`. However, this isn't too helpful if you want to remove a certain connection or if you just want to re-establish a connection that died. To help with this, there's the `ParameterType.LinkInfo` which, as the name implies, gives information about the links associated with your callback.

When you use a parameter type of `ParameterType.LinkInfo`, you'll get a dropdown menu where the user can select from live or dead links to leverage. When you select a current/dead link, the data that's sent down to your `create_tasking` function is the exact same as when you use the `ParameterType.ConnectionInfo` - i.e. information about the host, payload uuid, callback uuid, and the p2p c2 profile parameter information.
