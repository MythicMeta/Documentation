---
description: Token awareness and Token tasking
---

# Tokens

Mythic supports Windows tokens in two ways: tracking which tokens are viewable and tracking which tokens are usable by the callback. The difference here, besides how they show up in the Mythic interface, is what the agent can do with the tokens. The idea is that you can list out all tokens on a computer, but that doesn't mean that the agent has a handle to the token for use with tasking.&#x20;

### Tokens

As part of the normal responses sent back from the agent, there's an additional key you can supply, `tokens` that is a list of token objects that can be stored in the Mythic database. These will be viewable from the "Search" -> "Tokens" page, but are not leveraged as part of further tasking.

```
{ "action": "post_response",
    "responses": [
        {
            "task_id": "uuid here",
            "user_output": "got some tokens yo",
            "tokens": [
                {
                    "token_id": 18947, // required, agent generated
                    "host": "bob.com", // optional
                    "description": "", // optional
                    "user": "bob", // optional
                    "groups": "", //optional
                    "thread_id": 456, // optional
                    "process_id": 2345, // optional
                    "default_dacl": "", //optional p.TextField(null=True)
                    "session_id": 0, //optional p.IntegerField(null=True)
                    "restricted": false, //optional = p.BooleanField(null=True)
                    "capabilities": "", //optional = p.TextField(null=True)
                    "logon_sid": "", //optional = p.TextField(null=True)
                    "integrity_level_sid": 0, //optional = p.IntegerField(null=True)
                    "app_container_number": 0, //optional = p.IntegerField(null=True)
                    "app_container_sid": "", //optional = p.TextField(null=True)
                    "privileges": "", //optional = p.TextField(null=True)
                    "handle": 12345, //optional = p.IntegerField(null=True)
                }
            ]
        }
    ]
}
```

{% hint style="info" %}
`token_id` is simply a way for your callback to refer to the various tokens it interacts with. You'll use this `token_id` to register a token with your callback for use in subsequent tasking.
{% endhint %}

### Callback Tokens

If you want to be able to leverage tokens as part of your tasking, you need to register those tokens with Mythic and the callback. This can be done as part of the normal `post_response` responses like everything else. The key here is to identify the right token - specifically via the unique combination of token\_id and host.

```
{"action": "post_response",
    "responses": [
        {
            "task_id": "uuid here",
            "output": "now tracking token 12345",
            "callback_tokens": [
                {
                    "action": "add", // could also be "remove"
                    "host": "a.b.com", //optional - default to callback host if not specified
                    "token_id": 12345, // id 
                }
            ]
        }
    ]
}
                    
```

If the token `12345` hasn't been reported via the `tokens` key then it will be created and then associated with Mythic.

Once the token is created and associated with the callback, there will be a new dropdown menu  next to the tasking bar at the bottom of the screen where you can select to use the default token or one of the new ones specified. When you select a token to use in this way when issuing tasking, the `create_tasking` function's `task` object will have a new attribute, `task.token` that contains a dictionary of all the token's associated attributes. This information can then be used to send additional data with the task down to the agent to indicate which tokens should be used for the task as part of your parameters.&#x20;

Additionally, when getting tasks that have tokens associated with them, the `TokenId` value will be passed down to the agent as an additional field:\


```
{ "action": "get_tasking",
    "tasks": [
        {
            "command": "shell",
            "parameters": "whoami",
            "id": "uuid here",
            "timestamp": 1234567,
            "token": 12345
        }
    ]
}
```
