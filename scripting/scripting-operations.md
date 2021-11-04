# Scripting Operations

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.get_payloads()`:

```python
    async def get_current_operation_info(self) -> MythicResponse:
        """
        Gets information about the current operation for the user
        """

    async def get_all_operations(self) -> MythicResponse:
        """
        Gets information about all operations your operator can see
        """

    async def get_operation(self, operation: Operation) -> MythicResponse:
        """
        Gets information about the current user
        """

    async def add_or_update_operator_for_operation(
        self, operation: Operation, operator: Operator
    ) -> MythicResponse:
        """
        Adds an operator to an operation or updates an operator's view/block lists in an operation
        """

    async def remove_operator_from_operation(
        self, operation: Operation, operator: Operator
    ) -> MythicResponse:
        """
        Removes an operator from an operation
        """

    async def update_operation(self, operation: Operation) -> MythicResponse:
        """
        Updates information about an operation such as webhook and completion status
        """

    async def create_operation(self, operation: Operation) -> MythicResponse:
        """
        Creates a new operation and specifies the admin of the operation
        """
```

