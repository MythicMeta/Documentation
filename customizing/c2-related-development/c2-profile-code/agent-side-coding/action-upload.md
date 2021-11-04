---
description: Upload a file from Mythic to the target
---

# File upload (Mythic -> Agent)

## What does it mean to upload a file?

This section is specifically for uploading a file from the Mythic server to an agent. Because these messages aren't just to an agent directly (they can be routed through a p2p mesh), they can't just be as simple as a GET request to download a file. The file needs to be chunked and routed through the agents. This isn't specific to the `upload` command, this is for any command that wants to leverage a file from Mythic.

In general, there's a few steps that happen for this process (this can be seen visually on the [Message Flow](../../../../message-flow.md#file-uploads-from-mythic-greater-than-agent) page):

1. The operator issues some sort of tasking that has a parameter type of "File". You'll notice this in the Mythic user interface because you'll always see a popup for you to supply a file from your file system.&#x20;
2. In the Payload Type's corresponding command python file there is a function called `create_tasking` that handles the processing of tasks from users before handing them off to the database to be fetched by an agent. In this `create_tasking` function, the initial value of the parameter with type "File" will be the base64 blob of the file itself. This allows you to do nothing more and send down the full file as part of the `get_tasking` message for the agent. Now, while this is by far the easiest thing you can do, you lose the ability to track data, and if the file itself is rather large, then you lose the ability to use a different C2 mechanism or processing structure to fetch the file itself. Instead, we can track the file and send each chunk down separately.&#x20;
3. To register the file with Mythic, there's an RPC call we can do:

```javascript
async def create_tasking(self, task: MythicTask) -> MythicTask:
        original_file_name = json.loads(task.original_params)["file"]
        response = await MythicRPC().execute("create_file",
            file=base64.b64encode(task.args.get_arg("file")).decode(),
            saved_file_name=original_file_name,
            delete_after_fetch=False,
        )
        if response.status == MythicStatus.Success:
            task.args.add_arg("file", response.response["agent_file_id"])
        else:
            raise Exception("Error from Mythic: " + response.error)
        return task
```

When uploading a file, the `original_params` will be a JSON string where the parameter associated with your file parameter will have the name of the file you upload. This makes it handy from the operator perspective to see that they uploaded "evil.exe", but that data isn't necessary to pass down to the agent. The code on line 2 recovers the associated filename with the parameter name "file". In lines 3-7 we actually do the RPC call to register a file with mythic where `file` is the contents of the file and `delete_after_fetch` specifies if mythic should delete the file off disk once the agent has fully fetched it. Requiring you, the operator/developer, to manually register the file allows you to make checks and modifications on the file before registering it for mythic to use. Lines 8-9 is where we replace our base64 file contents in our tasking with the resulting file UUID.  The end result is that the agent gets a UUID for their parameter of type "File".

4\. The agent gets the tasking and sees that there's a file UUID it needs to pull. It sends an initial message to Mythic saying that it will be downloading the file in chunks of a certain size and requests the first chunk.If the agent is going to be writing the file to disk (versus just pulling down the file into memory), then the agent should also send `full_path` and `task_id`. This allows Mythic to track a new entry in the database with an associated task for uploads. The `full_path` key lives at the same level as `user_output`, `total_chunks`, etc in the `post_response` .

5\. The Mythic server gets the request for the file, makes sure the file exists and belongs to this operation, then gets the first chunk of the file as specified by the agent's `chunk_size` and also reports to the agent how many chunks there are.

6\. The Agent can now use this information to request the rest of the chunks of the file.

{% hint style="info" %}
The agent reporting back `full_path` is what allows Mythic to track the file in the "Active Callbacks" -> "Files" page as a file that has been written to disk. If you don't report back a `full_path` or have `full_path` as an empty string, then Mythic thinks that the file transfer only lived in memory and didn't touch disk. This is separate from reporting that a file was written to disk as part of artifact tracking on the Reporting Artifacts page.
{% endhint %}

It's not an extremely complex process, but it does require a bit more back-and-forth than a fire-and-forget style. The rest of this page will walk through those steps with more concrete code examples.

There is no expectation when doing uploads or downloads that the operator must type the absolute path to a file, that's a bit of a strict requirement. Instead, Mythic allows operators to specify relative paths and has an option in the upload action to specify the actual full path (this option also exists for downloading files so that the absolute path can be returned for better tracking). This allows Mythic to properly track absolute file system paths that might have the same resulting file name without an extra burden on the operator.

## Example (agent pull down):

Unlike with the files for loading, these files can (optionally) be pulled down multiple times (if you set `delete_after_fetch` as `True`, then the file won't exist on disk after the first fetch and thus can't be re-used). This is to prevent bloating up the server with unnecessary files.

An agent pulling down a file to the target is similar to downloading a file from the target to Mythic. The agent makes the following request to Mythic:

```javascript
Base64( CallbackUUID + JSON(
{
	"action": "post_response",
	"responses": [{
		"upload": {
			"chunk_size": 512000, //bytes of file per chunk
			"file_id": UUID, //the file specified to pull down to the target
			"chunk_num": #, // which chunk are we currently pulling down
			"full_path": "full path to uploaded file on target", //optional
		},
		"task_id": task_id // the associated task that caused the agent to pull down this file
}
))
```

The `full_path` parameter is helpful for accurate tracking. This allows an operator to be in the `/Temp` directory and simply call the upload function to the current directory, but allows Mythic to track the full path for easier reporting and deconfliction.&#x20;

{% hint style="info" %}
The `full_path` parameter is only needed if the agent plans to write the file to disk. If the agent is pulling down a file to load into memory, then there's no need to report back a `full_path`.
{% endhint %}

The agent gets back a message like:

```javascript
Base64( CallbackUUID + JSON(
{
	"action": "post_response",
	"responses": [ {
		"status": "success or error",
		"error": "error message if status is error, otherwise key not present",
		"total_chunks": #, // given the previous chunk size, the total num of chunks
		"chunk_num": #, //the current chunk number Mythic is returning
		"chunk_data": "base64_of_data" // the actual file data,
		"file_id": "file id that was requested",
		"task_id": "UUID of task" // task id that was presented in the request for tracking
		}
	]
}
))
```

This process repeats as many as times as is necessary to pull down all of the contents of the file.

{% hint style="danger" %}
If there is an error pulling down a file, the server will respond with as much information as possible and blank out the rest (i.e.: `{'action': 'post_response', 'responses': [ {'total_chunks': 0, 'chunk_num': 0, 'chunk_data': '', 'file_id': '', 'task_id': '', 'status': 'error', 'error': 'some error message'} ] }`)\
If the task\_id was there in the request, but there was an error with some other piece, then the task\_id will be present in the response with the right value.
{% endhint %}

### Files in the Tasking JSON

There's a reason that files aren't base64 encoded and placed inside the initial tasking blobs. Keeping the files tracked by the main Mythic system and used in a separate call allows the initial messages to stay small and allows for the agent and C2 mechanism to potentially cache or limit the size of the transfers as desired.

Consider the case of using DNS as the C2 mechanism. If the file mentioned in this section was sent through this channel, then the traffic would potentially explode. However, having the option for the agent to pull the file via HTTP or some other mechanism gives greater flexibility and granular control over how the C2 communications flow.
