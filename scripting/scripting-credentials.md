# Scripting Credentials

## What does this hook into?

Scripting tasking involves the following RESTful endpoints on an instance of `Mythic`. This means you need to create a new `Mythic` instance (i.e. `mythic = Mythic(username="blah" ...` ) and then call these functions like `mythic.get_all_tasks()`:

```python
    async def get_all_credentials(self) -> MythicResponse:
        """
        Get all of the credentials associated with the user's current operation
        :return:
        """

    async def create_credential(self, credential: Credential) -> MythicResponse:
        """
        Create a new credential associated with the user's current operation
        :return:
        """

    async def update_credential(self, credential: Credential) -> MythicResponse:
        """
        Create a new credential associated with the user's current operation
        :return:
        """
```

## How to create new credentials

Credential's `type` must be one of the following:&#x20;

* plaintext
* certificate
* hash
* key
* ticket
* cookie
* hex

To create a credential, you must supply a `type`, a `realm` (this is your domain if you're doing Windows), a `credential` (this is the actual credential value), and an `account` (who does this credential belong to). You can also optionally add a comment:

```python
cred = mythic_rest.Credential(type="plaintext", 
                              account="bob", 
                              credential="my cred", 
                              realm="my domain", 
                              comment="yo new credz")
    resp = await mythic.create_credential(cred)
    await mythic_rest.json_print(resp)
```
