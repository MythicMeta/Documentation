# File Upload Mythic->Agent

What happens when you want to transfer a file from Mythic -> Agent? There's two different options: tracking a file via a UUID and pulling down chunks or just sending the file as part of your tasking.

```mermaid
sequenceDiagram
    participant O as Operator
    participant M as Mythic
    participant P as PayloadType
    participant H as HTTP Container
    participant A as Agent
    O ->>+ M: Upload file1 with task to ./test
    M ->>+ P: Here's a task & file1
    P ->>+ M: Register this file1
    M -->>- P: Here's a file1 UUID
    P -->>- M: Tasking is ready
    M -->>- O: Tasking is Submitted
    A ->>+ H: Get Tasking
    H ->>+ M: Get Tasking
    M -->>- H: Here's your task
    H -->>- A: Here's your task
    A ->>+ H: Give me chunk1 of UUID with size X
    H ->>+ M: Give me chunk1 of UUID with size X
    M -->>- H: Here's chunk1, you have 3 total
    H -->>- A: Here's chunk1, you have 3 total
    A ->>+ H: Give me chunk2 of UUID with size X
    H ->>+ M: Give me chunk2 of UUID with size X
    M -->>- H: Here's chunk2, you have 3 total
    H -->>- A: Here's chunk2, you have 3 total
    A ->>+ H: Give me chunk3 of UUID with size X
    H ->>+ M: Give me chunk3 of UUID with size X
    M -->>- H: Here's chunk3, you have 3 total
    H -->>- A: Here's chunk3, you have 3 total
    A ->> A: Process file1
    A ->>+ H: file1 (UUID) is now at /abs/path/to/test
    H ->>+ M: file1 (UUID) is now at /abs/path/to/test
    M --> M: update file info
    M -->- H: updated tracking success
    H -->- A: updated tracking success
```

This is an example of an operator uploading a file, it getting processed at the Payload Type's `create_tasking` function where it tracks and registers the file within Mythic. Now the tasking has a UUID for the file rather than the file contents itself. This allows Mythic and the Agent to uniquely reference a file. The agent gets tasking, sees the file id, and submits more requests to fetch the file. Upon finally getting the full file, it resolves the relative upload path into an absolute path and sends an update back to Mythic to let it know that the file the operator said to upload to `./test` is actually at `/abs/pah/to/test` on the target host.

Conversely, you can opt to not track the file (or track the file within Mythic, but not send the UUID down to the agent). In this case, you can't easily reference the same instance of the file between the Agent and Mythic:

```mermaid
sequenceDiagram
    participant O as Operator
    participant M as Mythic
    participant P as PayloadType
    participant H as HTTP Container
    participant A as Agent
    O ->>+ M: Upload file2 with task to ./test2
    M ->>+ P: Here's a task & file2
    P -->>- M: Tasking is ready
    M -->>- O: Tasking is Submitted
    A ->>+ H: Get Tasking
    H ->>+ M: Get Tasking
    M -->>- H: Here's your task
    H -->>- A: Here's your task
    A ->> A: Process file2
    A ->>+ H: file2 is now at /abs/path/to/test2
    H ->>+ M: file2 is now at /abs/path/to/test2
    M --> M: WTF is this?
    M -->- H: wtf is this? no update
    H -->- A: wtf is this? no update
```

You're able to upload and transfer the file just fine, but when it comes to reporting back information on it, Mythic and the Agent can't agree on the same file, so it doesn't get updated. You might be thinking that this is silly, of course the two know what the file is, it was just uploaded. Consider the case of files being deleted or multiple instances of a file being uploaded.
