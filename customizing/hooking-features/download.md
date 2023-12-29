---
description: Download a file from the target to the Mythic server
---

# File Downloads (Agent -> Mythic)

## What does it mean to download a file?

This section is specifically for downloading a file from the agent to the Mythic server. Because these can be very large files that you task to download, a bit of processing needs to happen: the file needs to be chunked and routed through the agents.

In general, there's a few steps that happen for this process (visually this can be found on the [Message Flow](../../message-flow/#downloading-a-file-from-agent-greater-than-mythic) page):

1. The operator issues a task to download a file, such as `download file.txt`
2. This gets sent down to the agent in tasking, the agent locates the file, and determines it has the ability to read it. Now it needs to send the data back
3. The agent first gets the full path of the file so that it can return that data. That's a quality of life requirement so that operators don't need to supply the full path when specifying files, but so that Mythic can still properly track all files, especially ones that have the same name.
4. The agent then sends an initial message to the Mythic server indicating that it has a file to send. The agent specifies how many chunks it'll take, and which task this is for. If the agent specifies that there are `-1` total chunks, then Mythic will expect at some point for the agent to return a total chunk count so that Mythic knows the transfer is over. This can be helpful when the agent isn't able to seek the file length ahead of time.
5. The Mythic server then registers that file in the database so it can be tracked. This results in a file UUID. The Mythic server sends back this UUID so that the agent and Mythic can make sure they're talking about the same file for the actual transfer.
6. The agent then starts chunking up the data and sending it chunk by chunk. Each message will have chunk\_size amount of data base64 encoded, the file UUID from step 5, and which chunk number is being sent. Chunk numbers start at 1.
7. The Mythic server responds back with a successful acknowledgement that it received each chunk

It's not an extremely complex process, but it does require a bit more back-and-forth than a fire-and-forget style. This process allows Mythic to track how many chunks it'll take to download a file and how many have been downloaded so far. The rest of this page will walk through those steps with more concrete code examples.

## Example (agent response pt. 1):

When an agent is ready to transfer a file from agent to Mythic, it first needs to get the `full_path` of the file and determine how many chunks it'll take to transfer the file. It then creates the following structure:

```javascript
{"action": "post_response", "responses": [
    {
        "task_id": "UUID here",
        "download": {
            "total_chunks": 4, 
            "full_path": "/test/test2/test3.file", // optional full path to the file downloaded
            "host": "hostname the file is downloaded from", // optional
            "filename": "filename for Mythic/operator if full_path doesn't make sense", // optional
            "is_screenshot": false, //indicate if this is a file or screenshot (default is false)
            "chunk_size": 512000, // indicate chunk size if intending to send chunks out of order or paralellized
        }
    }
]}
```

The `host` field allows us to track if you're downloading files on the current host or remotely. If you leave this out or leave it blank (`""`), then it'll automatically be populated with the callback's hostname. Because you can use this same process for downloading files _and_ downloading screenshots from the remote endpoint in a chunked fashion, the `is_screenshot` flag allows this distinction. This helps the UI track whether something should be shown in the screenshot pages or in the files pages. If this information is omitted, then the Mythic server assumes it's a file (i.e. `is_screenshot` is assumed to be `false`). This message is what's sent as an [Action: post\_response](../payload-type-development/create\_tasking/agent-side-coding/action-post\_response.md) message.&#x20;

What if you don't know the total number of chunks ahead of time? No worries - register as normal but for the `total_chunks` field put a negative value. Later on, when you're sending chunks, you can add in your `total_chunks` and Mythic will simply update it on the fly.&#x20;

The `full_path` can be reported in any of the chunks and is an optional value. For example, if you collected a screenshot into memory and want to "download" it to Mythic, then there is no `full_path` to report back. In cases like this, you can specify a `filename` value that might make more sense (ex: `screenshot 1`, `monitor 2`, `lsass memory dump`, etc).

Mythic will respond with a file\_id:

```javascript
{"action": "post_response", "responses": [{
        "status": "success",
        "file_id": "UUID Here"
        "task_id": "task uuid here",
    }
]}
```

## Example (agent response pt. 2-n):

The agent sends back each chunk sequentially and calls out the file\_id its referring to along with the actual chunked data.

{% hint style="warning" %}
The `chunk_num` field is 1-based. So, the first chunk you send is `"chunk_num": 1`.
{% endhint %}

"Why on earth is `chunk_num` 1 based", you might be wondering. It's a legacy situation from Mythic 1.0 where everything was written in Python without proper struct tracking. This meant Mythic was having to do a lot of guess work for if keys weren't there or if agents were simply supplying "empty" or null fields that they weren't using as part of a message. This made it tricky to determine if an agent was referring to `chunk 0` or if they were simply setting that value to `0` because it wasn't being used (especially if a user tried to download a file that was 0 bytes in size). Starting real chunk data at `1` made it much easier to determine the scenario.&#x20;

Since then, Mythic was rewritten in Golang with stronger type checking, structs, and a slightly modified struct structure to help with all of this. Now it's a legacy thing so that everybody's agents don't have a breaking change.

{% hint style="info" %}
If your agent language is strongly typed or you need to supply all of the fields in every request, then for these additional file transfer messages, make sure the `total_chunks` field is set to `null`, otherwise Mythic will think you're trying to transfer another file.
{% endhint %}

```javascript
{"action": "post_response", "responses": [{
    {
        "task_id": "task uuid",
        "download": {
            "chunk_num": 1, 
            "file_id": "UUID From previous response", 
            "chunk_data": "base64_blob==",
            "chunk_size": 512000, // this is optional, but required if you're not sending it with the initial registration message and planning on sending chunks out of order
        }
    }
]}
```

For each chunk, Mythic will respond with a success message if all went well:

```javascript
{"action": "post_response", "responses": [{
    {
        "status": "success"
        "task_id": "task uuid here"
    }
]}
```

Once all of the chunks have arrived, the file will be downloadable from the Mythic server. Mythic can handle chunks out of order, but needs to know the `chunk_size` first. The `chunk_size` allows Mythic to seek the right spot in the file on disk before writing that chunk's data. `chunk_size` is **not** the size of the current chunk, Mythic can determine that much, but rather the size of every chunk the agent will try to read at a time. The last chunk is most likely not going to be the same size as the other chunks; because of this, Mythic needs to know the size of chunks in general in case it gets the last chunk first.
