# HTTP

## What is it?

The "HTTP" C2 profile speaks the exact same protocol as the Mythic server itself. All other C2 profiles will translate between their own special sauce back to this format. This profile has a docker container as well that you can start that uses a simple JSON configuration to redirect traffic on another port (with potentially different SSL configurations) to the main Mythic server.

## How does it work?

This container code starts a small Sanic web server that accepts messages on the specified port and proxies all connections to the `/api/v1.4/agent_message` endpoint within Mythic. This allows you to host the Mythic instance on port 7443 for example and expose the default HTTP profile on port 443 or 80.

Clicking the "Configure" button gives a few options for how to edit and interact with the profile.

#### Using SSL

If you want to use SSL with this profile, copy your SSL private key and cert into the `C2_Profiles/HTTP/c2_code` folder and update the `config.json` file to have the `key_path` and `cert_path` variables be the name of the private key and cert you copied over. You _have_ to copy the two files into this `c2_code` folder because the profile starts its own container which can't access files outside of the `c2_code` folder.

### Supported Payloads and Info

This sections allows you to see some information about the C2 profile, including sample configurations.

The name of a C2 profile cannot be changed once it's created, but everything else can change. The `Supported Payloads` shows which payload types can speak the language of this C2 profile.

### Profile Parameters

This dialog displays the current parameters associated with the C2 profile. These are the values you must supply when using the C2 profile to create an agent.

![Default C2 Profile parameters](<../.gitbook/assets/Screen Shot 2022-03-10 at 12.41.52 PM.png>)

There are a few things to note here:

* `randomize` - This specifies if you want to randomize the value that's auto-populated for the user.
* `format_string` - This is where you can specify how to generate the string in the hint when creating a payload. For example, setting `randomize` to `true` and a `format_string` of `\d{10}` will generate a random 10 digit integer.
  * This can be seen with the same `test` parameter in the above screenshot.
  * Every time you view the parameters, select to save an instance of the parameters, or go to create a new payload, another random instance from this format\_string will be auto-populated into that c2 profile parameter's hint field.
