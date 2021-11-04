---
description: Keystrokes are sent back from the agent to the Mythic server
---

# Keylog

## Example (agent response):

```javascript
{
    "task_id": "task uuid here",
    "keylogs": [
        {
            "user": "its-a-feature", 
            "window_title": "Notepad - Untitled", 
            "keystrokes": "my password is zer0c00l"
        }
    ]
}
```

## Walkthrough:

Agents can report back keystrokes at any time. There are three components to a keystroke report:

* `user` - the user that is being keylogged
* `window_title` - the title of the window to which the keystrokes belong
* `keystrokes` - the actual recorded keystrokes

Having the information broken out into these separate pieces allows Mythic to do grouping based on the user and window\_title for easier readability.

If the agent doesn't know the user or the window\_title fields, they should still be included, but can be empty strings. If empty strings are reported for either of these two fields, they will be replaced with "UNKNOWN" in Mythic.

### Multiple users/windows

What happens if you need to send keystrokes for multiple users/windows?

```python
{
    "action": "post_response",
    "responses": [
        {
            "task_id": "task uuid here",
            "keylogs": [
                {
                    "user": "its-a-feature", 
                    "window_title": "Notepad - Untitled", 
                    "keystrokes": "my password is zer0c00l"
                },
                {
                    "user": "its-a-feature", 
                    "window_title": "Notepad - Untitled", 
                    "keystrokes": "my password is zer0c00l"
                }
                ,{
                    "user": "its-a-feature", 
                    "window_title": "Notepad - Untitled", 
                    "keystrokes": "my password is zer0c00l"
                }
            ]
        }
    ]
}
```
