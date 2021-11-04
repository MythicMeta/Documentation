# Scripting Payload Types

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.download_file()`:

```python
async def get_payloadtypes(self) -> MythicResponse:
        """
        Get all payload types registered with Apfell
        :return:
        """

    async def get_payloadtype(
        self, payload_type: Union[PayloadType, Dict]
    ) -> MythicResponse:
        """
        Get information about a specific payload type
        :param payload_type:
            if using PayloadType class, the following must be set:
                ptype
        :return:
        """

    async def get_payloadtype_commands(
        self, payload_type: Union[PayloadType, Dict]
    ) -> MythicResponse:
        """
        Get the commands registered for a specific payload type
        :param payload_type:
            if using PayloadType class, the following must be set:
                ptype
        :return:
        """
```
