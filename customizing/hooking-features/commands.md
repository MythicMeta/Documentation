---
description: >-
  This page discusses how to register that commands are loaded/unloaded in a
  callback
---

# Commands

## Example Agent Response

```javascript
{
    "task_id": "task uuid here",
    "user_output": "some user output here",
    "commands": [
        {
            "action": "add",
            "cmd": "shell"
        },
        {
            "action": "add",
            "cmd": "jsimport"
        }
    ]
}
```

## Walkthrough

It's a common feature for agents to be able to load new functionality. Even within Mythic, you can create agents that only start off with a base set of commands and more are loaded in later. Using the `commands` keyword, agents can report back that commands are added (`"action": "add"`) or removed (`"action": "remove"`).

This is easily visible when interacting with an agent. When you start typing a command, you'll see an autocomplete list appear above the command prompt with a list of commands that include what you've typed so far. When you load a new command and register it back with mythic in this way, that new command will also appear in that autocomplete list.
