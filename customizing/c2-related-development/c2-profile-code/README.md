---
description: How to design C2 profile code for Mythic
---

# C2 Profile Code

There are two components for C2 code: agent side coding and server side coding.

#### Agent Side Coding

This is the component that goes into your agent and handles hooking into the Mythic features, does chunking for files, reports back artifacts, keystrokes, etc.

#### Server Side Coding

This is the component that lives within the C2 Docker container and actually translates the C2 requests from special sauce to Mythic's HTTP + JSON.
