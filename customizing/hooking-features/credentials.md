---
description: Agents can report back credentials they discover
---

# Credentials

## Example (agent response):

```javascript
{
    "task_id": "task uuid here",
    "user_output": "some user output here",
    "credentials": [
        {
            "credential_type": "plaintext",
            "realm": "spooky.local",
            "credential": "SuperS3Cr37",
            "account": "itsafeature"
        }
    ]
}
```

## Walkthrough:

The agent can report back multiple credentials in a single response. The `credential_type` field represents the kind of credential and must be one of the following:

* plaintext
* certificate
* hash
* key
* ticket
* cookie

The other fields are pretty straightforward, but they must all be provided for each credential. There is one optional field that can be specified here: `comment`. You can do this manually on the credentials page, but you can also add comments to every credential to provide a bit more context about it.
