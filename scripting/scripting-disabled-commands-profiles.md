# Scripting Disabled Commands Profiles

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.get_all_disabled_commands_profiles()`:

```python
    async def get_all_disabled_commands_profiles(self) -> MythicResponse:
        """
        Get all of the disabled command profiles associated with Mythic
        :return:
        """

    async def create_disabled_commands_profile(
        self, profile: DisabledCommandsProfile
    ) -> MythicResponse:
        """
        Create a new disabled command profiles associated with Mythic
        :return:
        """

    async def update_disabled_commands_profile(
        self, profile: DisabledCommandsProfile
    ) -> MythicResponse:
        """
        Create a new disabled command profiles associated with Mythic
        :return:
        """

    async def update_disabled_commands_profile_for_operator(
        self,
        profile: Union[DisabledCommandsProfile, str],
        operator: Operator,
        operation: Operation,
    ) -> MythicResponse:
        # async def add_or_update_operator_for_operation(self, operation: Operation, operator: Operator)
        if isinstance(profile, mythic_rest.DisabledCommandsProfile):
            operator.base_disabled_commands = profile.name
        else:
            operator.base_disabled_commands = profile
        resp = await self.add_or_update_operator_for_operation(operation, operator)
        return resp
```

