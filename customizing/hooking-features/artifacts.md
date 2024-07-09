---
description: Agent reports new artifacts created on the system or network
---

# Artifacts

## Example (user tasking):

Any command is able to reply with its own artifacts that are created along the way. The following response can be returned as a separate C2 message or as part of the command's normal output.

{% hint style="info" %}
The following response is part of the normal agent response. So, it is base64 encoded and put in the normal response format
{% endhint %}

## Example (agent response):

```javascript
{
    "task_id": "task uuid here",
    "user_output": "some user output here",
    "artifacts": [
        {
            "base_artifact": "Process Create",
            "artifact": "sh -c whoami",
            "needs_cleanup": false, // optional, defaults to false
            "resolved": false, // optional, defaults to false
        },
        {
            "base_artifact": "File Write",
            "artifact": "/users/itsafeature/Desktop/notmalware.exe",
            "needs_cleanup": true, // optional, defaults to false
            "resolved": false, // optional, defaults to false
        }
    ]
}
```

## Walkthrough:

Agents can report back their own artifacts they create at any time. They just include an `artifacts` keyword with an array of the artifacts. There are two components to this:

1. `base_artifact` is the type of base artifact being reported. If this base\_artifact type isn't already captured in the "Global Configurations" -> "Artifact Types" page, then this `base_artifact` value will be created.
2. `artifact` is the actual artifact being created. This is a free-form field.
3. `needs_cleanup` - this is an optional field that indicates if this artifact will need to be cleaned up at some point
4. `resolved` - this is an optional field that indicates if the artifact is already cleaned up

Artifacts created this way will be tracked in Artifacts page (click the fingerprint icon at the top)
