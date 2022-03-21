# Screenshots

Mythic has a special page specifically for viewing screenshots by clicking the camera icon at the top of any of the pages.&#x20;

When it comes to registering screenshots with Mythic, the process is almost identical to  [download.md](download.md "mention"); however, we set the `is_screenshot` flag to `true` in the `download` portion of the message:

```json
{"action": "post_response", "responses": [
    {
        "task_id": "UUID here",
        "download": {
            "total_chunks": 4, 
            "full_path": "/test/test2/test3.file" // full path to the file downloaded
            "host": "hostname the file is downloaded from"
            "is_screenshot": true //indicate if this is a file or screenshot
        }
    }
]}
```

