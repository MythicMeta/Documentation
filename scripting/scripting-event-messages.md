# Scripting Event Messages

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.get_all_disabled_commands_profiles()`:

```python
    async def get_all_event_messages(self) -> MythicResponse:
        """
        Get all of the event messages associated with Mythic for the current operation that are not deleted
        :return:
        """

    async def create_event_message(self, message: EventMessage) -> MythicResponse:
        """
        Create new event message for the current operation
        :return:
        """

    async def update_event_message(self, message: EventMessage) -> MythicResponse:
        """
        Update event message for the current operation
        :return:
        """

    async def remove_event_message(self, message: EventMessage) -> MythicResponse:
        """
        Update event message for the current operation
        :return:
        """
```
