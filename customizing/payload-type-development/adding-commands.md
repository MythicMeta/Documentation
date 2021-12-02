# Adding Commands

## Adding New Commands

So, you want to add a new command to a Payload Type. What does that mean, where do you go, what all do you have to do?

Luckily, the Payload Type containers are the source of truth for everything related to them, so that's the only place you'll need to edit. If your payload type uses its own custom message format, then you might also have to edit your associated translation container, but that's up to you.

1. Make a new `.py` file in `Payload_Types/[agent name/mythic/agent_functions`.
2. This new file should match the requirements of the rest of the [commands](commands.md#what-do-commands-track)
3. Once you're done making edits, restart your payload type container via: `./mythic-cli payload start [payload type name]`. This will restart just that one payload type container, reloading the python files automatically, and re-syncing the data with Mythic.
