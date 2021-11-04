# Next Release

## What's planned for the next release (2.1)?

* [ ] Tracking of Sleep information in the UI somehow
* [ ] Python script to interact with tasking/websockets/responses
  * [x] Write the code
  * [ ] Document its usage
* [x] Stricter C2 profile parameter checks when building payloads
* [ ] Move KillDate out of C2 profile parameter into Payload BuildParameter
* [ ] Documentation on how to support 3rd party agents into Mythic
  * [ ] Agent Installer script to pull in 3rd party agents via GitHub into your Mythic instance
  * [ ] Documentation on what format the GitHub repo/release must follow
    * [ ] Should include: Payload\_Type code, C2 Profile code (if any), Documentation Docker container code for all pieces
* [x] Display badge count for unresolved warning events in current operation
* [x] Allow users to send messages as "warning" messages that need resolution
* [x] Restrict downloading of payloads to the same "allow block" as the /login, /auth, /register endpoints

## How can I submit something to be added in the next release?

* Open up a feature request on the Mythic github page (https://github.com/its-a-feature/Mythic)
* Ask for it in the #mythic slack channel in the public Bloodhound slack

