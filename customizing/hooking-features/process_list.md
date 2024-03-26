---
description: Unified process listing across multiple callbacks for a single host
---

# Process Browser

### Command Component

The supported ui features flag on the command that does this tasking needs to have the following set:`supported_ui_features = ["process_browser:list"]` if you want to be able to issue a process listing from the UI process listing table. If you don't care about that, then you don't need that feature set for your command.

### Why a Unified Process List per Host

There are many instances where you might have multiple agents running on a single host and you run common tasking like process lists over and over and over again. You often do this because the tasking has scrolled out of view, maybe it's just stale information, or maybe you couldn't quite remember which callback actually had that task. This is where the unified process listing comes into play.

With a special format for process listing, Mythic can track all the different process lists together for a single host. It doesn't matter which host you ran the task on, as long as you pull up the process\_list view for that host, all of the tasks will be available and viewable.

### Output Format

Naturally, this has a special format for us to make it the most useful. Like almost everything else in Mythic, this requires structured output for each process we want the following:

```python
 {"action": "post_response", "responses": [
   {
   "task_id": "uuid",
   "processes": [
       {
        "process_id": 12345, 
        "name": "evil.exe"
        "host": "a.b.com", //optional
        "parent_process_id": 1234, //optional
        "architecture": "x64", // optional
        "bin_path": "C:\\Users\\bob\\Desktop\\evil.exe", // optional
        "user": "bob", // optional
        "command_line": "C:\\Users\\bob\\Desktop\\evil.exe -f test.txt -thread 12", // optional
        "integrity_level": 3, // optional 
        "start_time": unix epoch time in milliseconds, //optional
        "description": "not malware", // optional
        "signer": "Bob's software co", // optional
        "protected_process_level": 0, // optional
        "update_deleted": false, // optional - setting this to true tells Mythic to mark any process not returned in this process array as deleted
        ** // any other fields you want, they all end up in the metadata field within the database
        } 
    ]
  }
]}
```

All that's needed is an array of all the processes with the above information in the `processes` field of your `post_response` action. That allows Mythic to create a process hierarchy (if you supply both `process_id` and `parent_process_id`) and a sortable/filterable table of processes. The above example shows a `post_response` with one response in it. That one response has a `processes` field with an array of processes it wants to report.&#x20;

Any field that ends with `_time` expects the value to be an int64 of unix epoch time in milliseconds. You're welcome to supply any additional field you want about a process - it all gets aggregated together and provided as part of the "metadata" for the process that you can view in the UI in a nice table listing.

For example, a macOS agent might report back signing flags and entitlements and a windows agent might report back integrity level and user session id.
