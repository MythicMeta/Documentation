# Artifacts

## What is it?

Artifacts track potential indicators of compromise and other notable events occurring throughout an operation.

## Where is it?

A list of all current artifacts can be found on the "Reporting" -> "Reporting Artifacts" page from the top navigational bar.

![Artifact reporting Page](<../.gitbook/assets/Screen Shot 2020-08-20 at 11.03.09 AM.png>)

## How to use it?

This page tracks all of the artifacts automatically created by executing tasks, those reported by agents, and those manually entered. This should provide a good idea for both defenders and red teamers about the artifacts left behind during an operation and should help with deconfliction requests.

The entire artifact database for the current operation can be dumped in JSON format via the "Export Artifacts" button at the bottom of the screen. Users can manually add artifacts and tie them to specific tasks or just keep them broadly applied.

### When is it updated?

Artifacts are created in a few different ways:

1. A command's tasking automatically creates an artifact.
2. A user manually adds in a new artifact
3. An agent reports back a new artifact in an ad-hoc fashion
