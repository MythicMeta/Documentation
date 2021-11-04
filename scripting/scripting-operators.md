# Scripting Operators

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.get_payloads()`:

```python
    async def get_self(self) -> MythicResponse:
        """
        Gets information about the current user
        """

    async def get_operator(self, operator: Operator) -> MythicResponse:
        """
        Gets information about the current user
        """

    async def create_operator(self, operator: Operator) -> MythicResponse:
        """
        Creates a new operator with the specified username and password.
        If the operator name already exists, just returns information about that operator.
        """

    async def update_operator(self, operator: Operator) -> MythicResponse:
        """
        Updates information about the specified operator.
        """
```

