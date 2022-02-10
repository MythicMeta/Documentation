# Agent Side Coding

This section talks about the different components for creating messages _from_ the agent _to_ a C2 docker container and how those can be structured within a C2 profile. Specifically, this goes into the following components:

* [Uploading](../../../hooking-features/action-upload.md) Files
* How [agent messages](agent-message-format.md) are formatted
* How to perform [initial checkins](initial-checkin.md) and do encrypted [key exchanges](initial-checkin.md#encrypted-key-exchange-checkins)
* How to [Get Tasking](action\_get\_tasking.md)
* How to [Post Responses](action-post\_response.md)

Another major component of the agent side coding is the actual C2 communications piece within your agent. This piece is how your agent actually implements the C2 components to do its magic.

Every C2 profile has zero or more C2 Parameters that go with it. These describe things like callback intervals, API keys to use, how to format web requests, encryption keys, etc. These parameters are specific to that C2 profile, so any agent that "speaks" that c2 profile's language will leverage these parameters. If you look at the parameters in the UI, you'll see:

* `Name` - When creating payloads or issuing tasking, you will get a dictionary of `name` -> `user supplied value` for you to leverage. This is a unique key per C2 profile (ex: `callback_host`)
* `description` - This is what's presented to the user for the parameter (ex: `Callback host or redirector in URL format`)
* `default_value` - If the user doesn't supply a value, this is the default one that will be used
* `verifier_regex` - This is a regex applied to the user input in the UI for a visual cue that the parameter is correct. An example would be `^(http|https):\/\/[a-zA-Z0-9]+` for the `callback_host` to make sure that it starts with http:// or https:// and contains at least one letter/number.
* `required` - Indicate if this is a required field or not.
* `randomized` - This is a boolean indicating if the parameter should be randomized each time. This comes into play each time a payload is generated with this c2 profile included. This allows you to have a random value in the c2 profile that's randomized for each payload (like a named pipe name).
* `format_string` - If `randomized` is `true`, then this is the regex format string used to generate that random value. For example, `[a-z0-9]{8}-[a-z0-9]{4}-[a-z0-9]{4}-[a-z0-9]{4}-[a-z0-9]{12}` will generate a UUID4 each time.
