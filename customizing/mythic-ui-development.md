# Mythic UI Development

## Mythic UI

The current Mythic UI is split into two parts:

1. Normally, when using mythic, `Mythic/mythic-react-docker` and the `mythic_react` container serves a compiled version of the React UI.
2. For development, the `Mythic/MythicReactUI` folder holds the source code that gets built.

### Modifying the Mythic UI

To load up the source code for active development, set the `Mythic/.env` variable `mythic_react_debug` to `true` and then run `sudo ./mythic-cli start mythic_react`. This will tear down the current `mythic_react` container and re-build it with nodejs.

At this point, any changes you make to the source code in `Mythic/MythicReactUI` will trigger automatic re-builds on the UI for easy development.

### Building the Mythic UI

When you're done with all of your changes, run `sudo ./mythic-cli build_ui` and Mythic will automatically build the UI and place it in the `mythic-react-docker` folder. At this point, if you set the `mythic_react_debug` back to `false` and run `./mythic-cli start mythic_react`, you'll see the UI exactly as somebody else would with a fresh install of Mythic.
