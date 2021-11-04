---
description: >-
  This page will keep a running set of frequently asked questions or
  troubleshooting tips
---

# FAQ / Troubleshooting Tips

## Tried starting Mythic, but the mythic\_postgres container keeps restarting.

This likely means that the postgres container can't bind to the port it needs. Check to make sure there isn't already something listening locally on 5432. If you're using Kali Linux, it's likely that metasploit is already using that port for its postgres database.

## Started the default container, created a payload, but it's not connecting back.

There a few common possibilities:

* **Wrong IP/Port/Protocol somewhere**. All of the C2 profile docker containers simply accept connections from agents, remove any special C2 magic that you're using, and redirect the actual agent message to your Mythic instance.&#x20;
  * Default Mythic UI is on port 7443 with SSL. The `HTTP` profile defaults to HTTP (no SSL) and port 80. When creating the callback host for your payload, specify either `http://mythicIP` to connect back directly through the C2 container or specify your redirector and make sure your redirector is set to go back to the right IP:Port combination.
* **C2 Server isn't running**. All C2 components are composed of 2 pieces - the Docker container and the service running within it. Go to the "Global Configurations" -> "C2 Profiles" page and check the corresponding row for your profile. If there's a grey button saying to "Start Internal Server", click it to issue the start command to the Docker container.
* **C2 Server is running, but no callbacks**. Depending on the C2 Profile you're using, there's likely a True/False indicator for debugging information (this is the case with the built-in profiles). Make sure that is set to True in the JSON configuration (restart your C2 profile if necessary). On the "Global Configurations" -> "C2 Profiles" page, when the C2 Profile is running there will be a red button with a dropdown. Click that dropdown and select to "View Stdout/Stderr". This will show you the latest un-read messages from the C2 profile debugging information. This will give some information about if the Docker container is even getting the connections or not.

## Trying to start Mythic, but getting a memory space error?

We've seen this a few times when people are on smaller VMs or pretty full VMs and are trying to include Poseidon. The Poseidon container is about 3.5GB currently since it contains the SDKs and information for cross compiling golang for macOS and Linux. We're working on making that container smaller and removing all the pieces we aren't using, but the easiest step here is to free up some room.



