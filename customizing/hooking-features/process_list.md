---
description: Unified process listing across multiple callbacks for a single host
---

# Process\_List

### Command Component

The supported ui features flag on the command that does this tasking needs to have the following set:`supported_ui_features = ["process_browser:list"]` .

### Why a Unified Process List per Host

There are many instances where you might have multiple agents running on a single host and you run common tasking like process lists over and over and over again. You often do this because the tasking has scrolled out of view, maybe it's just stale information, or maybe you couldn't quite remember which callback actually had that task. This is where the unified process listing comes into play.

With a special format for process listing, Mythic can track all the different process lists together for a single host. It doesn't matter which host you ran the task on, as long as you pull up the process\_list view for that host, all of the tasks will be available and viewable.

### Output Format

Naturally, this has a special format for us to make it the most useful. Like almost everything else in Mythic, this requires structured output for each process we want the following:

```python
 {"action": "post_response", "responses": [
   {"processes": [
       {
        "process_id": pid, 
        "host": "a.b.com", //optional
        "architecture": "x64", //optional
        "name": "lol.exe", //optional
        "user": "its-a-feature", //optional
        "bin_path": "C:\\whatever", //optional
        "parent_process_id": ppid, //optional
        "command_line": "command line if you can get it", //optional
        "integrity_level": 3, //optional
        "start_time": unix epoch time in miliseconds, //optional
        "description": "if it has one", //optional
        "signer": "signing information if you can get it", //optional
    } 
    ]
  }
]}
```

All that's needed is an array of all the processes with the above information in the `processes` field of your `post_response` action. That allows Mythic to create a process hierarchy (if you supply both `process_id` and `parent_process_id`) and a sortable/filterable table of processes. The above example shows a `post_response` with one response in it. That one response has a `processes` field with an array of processes it wants to report.

This new view also allows you to `diff` two different process listing outputs to see what processes were added or removed.
