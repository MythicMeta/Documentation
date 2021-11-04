# Process Response

## What is Process Response?

Within a `Command` class, there are two functions - `create_tasking` and `process_response`. As the names suggest, the first one allows you to create and manipulate a task before an agent pulls it down, and the second one allows you to process the response that comes back. If you've been following along in development, then you know that Mythic supports many different fields in its `post_response` action so that you can automatically create artifacts, register keylogs, manipulate callback information, etc. However, all of that requires that your agent format things in a specific way for Mythic to pick them up and process. That can be tiring.&#x20;

### Enter \`process\_response\`

The `process_response` function takes in one argument class that contains two pieces of information: The `task` that generated the response in the first place, and the `response` that was sent back from the agent. Now, there's a specific `process_response` keyword you have to send for mythic to shuttle data off to this function instead of processing it normally. When looking at a `post_response` message, it's structured as follows:

```python
{
    "action": "post_response",
    "responses": [
        { 
            "task_id": "some uuid",
            "process_response": {"myown": "data format"},
            // all of the other fields you want to leverage
        }
    ]
}
```

Now, anything in that `process_response` key will get sent to the `process_response` function in your Payload Type container.

Some caveats:

* You can send any data you want in this way and process it however you want. In the end, you'll be doing RPC calls to Mythic to register the data
* Not all things make sense to go this route. Because this is an async process to send data to the container for processing, this happens asynchronously and in parallel to the rest of the processing for the message. For example, if your message has _just_ the `task_id` and a `process_container` key, then as soon as the data is shipped off to your `process_response` function, Mythic is going to send the all clear back down to the agent and say everything was successful. It doesn't wait for your function to finish processing anything, nor does it expect any output from your function.
  * We do this sort of separation because your agent shouldn't be waiting on the hook for unnecessary things. We want the agent to get what it wants as soon as possible so it can go back to doing agent things.&#x20;
* Some functionality like SOCKS and file upload/download don't make sense for the `process_response` functionality because the agent needs the response in order to keep functioning. Compare this to something like registering a keylog, creating an artifact, or providing some output to the user which the agent tends to think of in a "fire and forget" style. These sorts of things are fine for async parallel processing with no response to the agent.

The function itself is really simple:

```python
async def process_response(self, response: AgentResponse):
    #resp = await MythicRPC().execute("create_callback_token", host=response.task.callback.host,
    #                                                     add=response.response, remove=[])
    pass
```

where `response` has two fields:

* `response.task` is the task you'd expect from the OPSEC checking or `create_tasking` components
* `response.response` is the data you put in the `process_response` key.

You have full access to all of the RPC methods to Mythic from here just like you do from the other functions.
