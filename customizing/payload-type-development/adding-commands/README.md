# 3. Adding Commands

## Adding New Commands

So, you want to add a new command to a Payload Type. What does that mean, where do you go, what all do you have to do?

Luckily, the Payload Type containers are the source of truth for everything related to them, so that's the only place you'll need to edit. If your payload type uses its own custom message format, then you might also have to edit your associated translation container, but that's up to you.

{% tabs %}
{% tab title="Python" %}
Make a new `.py` file with your command class and make sure it gets imported before `mythic_container.mythic_service.start_and_run_forever` is called so that the container is aware of the command before syncing over.

This new file should match the requirements of the rest of the [commands](commands.md#what-do-commands-track)

Once you're done making edits, restart your payload type container via: `./mythic-cli start [payload type name]`. This will restart just that one payload type container, reloading the python files automatically, and re-syncing the data with Mythic.
{% endtab %}

{% tab title="Golang" %}
Make a new `.go` file with your new command struct instance. You can either do this as part of an `init` function so it gets picked up automatically when the package/file is imported, or you can have specific calls that initialize and register the command.&#x20;

Eventually, run `agentstructs.AllPayloadData.Get("agent name").AddCommand` so that the Mythic container is aware that the command exists. Make sure this line is executed before your `MythicContainer.StartAndRunForever` function call.

This new file should match the requirements of the rest of the [commands](commands.md#what-do-commands-track)

Once you're done making edits, restart your payload type container via: `./mythic-cli build [payload type name]`. This will rebuild and restart just that one payload type container and re-syncing the data with Mythic.
{% endtab %}
{% endtabs %}
