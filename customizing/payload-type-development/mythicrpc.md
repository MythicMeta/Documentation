# MythicRPC

## What is MythicRPC

MythicRPC provides a way to execution functions against Mythic and Mythic's database programmatically from within your command's tasking files.

## Where is MythicRPC

MythicRPC lives as part of the `mythic_container` PyPi package (and `github.com/MythicMeta/MythicContainer` GoLang package) that's included in all of the `itsafeaturemythic` Docker images. This PyPi package uses rabbitmq's RPC functionality to execute functions that exist within Mythic.

The full list of commands can be found here: [https://github.com/MythicMeta/MythicContainerPyPi/tree/main/mythic\_container/MythicGoRPC](https://github.com/MythicMeta/MythicContainerPyPi/tree/main/mythic\_container/MythicGoRPC) for Python and [https://github.com/MythicMeta/MythicContainer/tree/main/mythicrpc](https://github.com/MythicMeta/MythicContainer/tree/main/mythicrpc) for GoLang.
