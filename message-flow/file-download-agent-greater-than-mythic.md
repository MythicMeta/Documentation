# File Download Agent->Mythic

When you're downloading a file from the Agent to Mythic (such as a file on disk, a screenshot, or some large piece of memory that you want to track as a file), you have to indicate in some way that this data is specific to a file and not destined to be part of the information displayed to the user. The way this works is pretty much the inverse of what happens for uploads. Specifically, an agent has a file it wants to transfer, so it tells Mythic "i have data, i'm gonna send it as X chunks of size Y, can you give me a UUID so we can track this". Mythic tracks the data and gives back a UUID. Now the agent sends each chunk individually and Mythic can track it. This allows a single task to be able to send back multiple files concurrently or sequentially and still track it all.

```mermaid
sequenceDiagram
    participant O as Operator
    participant M as Mythic
    participant P as PayloadType
    participant H as HTTP Container
    participant A as Agent
    O ->>+ M: Download test
    M ->>+ P: Here's a task
    P -->>- M: Tasking is ready
    M -->>- O: Tasking is Submitted
    A ->>+ H: Get Tasking
    H ->>+ M: Get Tasking
    M -->>- H: Here's your task
    H -->>- A: Here's your task
    A ->>+ H: Track new file, 5 total_chunks
    H ->>+ M: Track new file, 5 total_chunks
    M ->> M: Create File in DB
    M -->>- H: Here's a file UUID
    H -->>- A: Here's a file UUID
    loop send chunks 1-5
        A ->>+ H: Here's chunkX for file UUID
        H ->>+ M: Here's chunkX for file UUID
        M ->> M: Stores data to disk
        M -->>- H: Success
        H -->>- A: Success
    end
    A ->>+ H: file (UUID) is at /abs/path/to/test
    H ->>+ M: file (UUID) is at /abs/path/to/test
    M --> M: update file info
    M -->- H: updated tracking success
    H -->- A: updated tracking success
```
