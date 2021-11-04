# Scripting API Tokens

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.get_payloads()`:

```python
    async def get_apitokens(self) -> MythicResponse:
        """
        Gets all of the user's API tokens in a List
        :return:
        """

    async def create_apitoken(self, token_type="User") -> MythicResponse:
        """
        Creates an API token for the user
        :param token_type:
            must be either "User" or "C2"
        :return:
        """

    async def remove_apitoken(self, apitoken: Union[APIToken, Dict]) -> MythicResponse:
        """
        Removes the specified API token and invalidates it going forward
        :param apitoken:
            if using the APIToken class, the following must be set:
                id
        :return:
        """
```
