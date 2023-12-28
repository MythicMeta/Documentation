# Turning a VM into a Container

There are scenarios in which you need a Mythic container for an agent, but you can't (or don't want) to use the normal docker containers that Mythic uses. This could be for reasons like:

* You have a custom build environment that you don't want to recreate
* You have specific kernel versions or operating systems you're wanting to develop with

So, to leverage your own custom VM or physical computer into a Mythic recognized container, there are just a few steps.

{% hint style="warning" %}
External agents need to connect to `mythic_rabbitmq` in order to send/receive messages. They also need to connect to the `mythic_server` to transfer files and potentially use gRPC. By default, these container is bound on localhost only. In order to have an external agent connect up, you will need to adjust this in the `Mythic/.env` file to have `RABBITMQ_BIND_LOCALHOST_ONLY=false` and `MYTHIC_SERVER_BIND_LOCALHOST_ONLY=false` and restart Mythic (`sudo ./mythic-cli restart`).&#x20;
{% endhint %}

1. Install python 3.10+ (or Golang 1.21) in the VM  or on the computer
2. `pip3 install mythic-container` (this has all of the definitions and functions for the container to sync with Mythic and issue RPC commands). Make sure you get the right version of this PyPi package for the version of Mythic you're using ([#current-payloadtype-versions](container-syncing.md#current-payloadtype-versions "mention")). Alternatively, `go get -u github.com/MythicMeta/MythicContainer` for golang.
3. Create a folder on the computer or VM (let's call it path `/pathA`). Essentially, your `/pathA` path will be the new `InstalledServices/[agent name]` folder. Create a sub folder for your actual agent's code to live, like `/pathA/agent_code`. You can create a Visual Studio project here and simply configure it however you need.
4. Your command function definitions and payload definition are also helpful to have in a folder, like `/pathA/agent_functions`.
5.  Edit the `/pathA/rabbitmq_config.json` with the parameters you need\


    <pre><code>{
      "rabbitmq_host": "127.0.0.1",
      "rabbitmq_password": "PqR9XJ957sfHqcxj6FsBMj4p",
      "mythic_server_host": "127.0.0.1",
      "webhook_default_channel": "#mythic-notifications",
      "debug_level": "debug",
      "rabbitmq_port": 5432,
      "mythic_server_grpc_port": 17444,
    <strong>  "webhook_default_url": "",
    </strong><strong>  "webhook_default_callback_channel": "",
    </strong>  "webhook_default_feedback_channel": "",
      "webhook_default_startup_channel": "",
      "webhook_default_alert_channel": "",
      "webhook_default_custom_channel": "",
    }
    </code></pre>

    1. the `mythic_server_host` value should be the IP address of the main Mythic install
    2. the `rabbitmq_host` value should be the IP address of the main Mythic install unless you're running rabbitmq on another host.
    3. You'll need the password of rabbitmq from your Mythic instance. You can either get this from the `Mythic/.env` file, by running `sudo ./mythic-cli config get rabbitmq_password`, or if you run `sudo ./mythic-cli config payload` you'll see it there too.
6. External agents need to connect to `mythic_rabbitmq` in order to send/receive messages. By default, this container is bound on localhost only. In order to have an external agent connect up, you will need to adjust this in the `Mythic/.env` file to have `RABBITMQ_BIND_LOCALHOST_ONLY=false` and restart Mythic (`sudo ./mythic-cli restart`). You'll also need to set `MYTHIC_SERVER_BIND_LOCALHOST_ONLY=false`.
7. In the file where you define your payload type is where you define what it means to "build" your agent.
8. Run `python3.10 main.py` and now you should see this container pop up in the UI
9. If you already had the corresponding payload type registered in the Mythic interface, you should now see the red text turn green.

You should see output similar to the following:

```
itsafeature@spooky my_container % python3 main.py    
INFO 2023-04-03 21:17:10,899 initialize  29  : [*] Using debug level: debug
INFO 2023-04-03 21:17:10,899 start_services  267 : [+] Starting Services with version v1.0.0-0.0.7 and PyPi version 0.2.0-rc9

INFO 2023-04-03 21:17:10,899 start_services  270 : [*] Processing webhook service
INFO 2023-04-03 21:17:10,899 syncWebhookData  261 : Successfully started webhook service
INFO 2023-04-03 21:17:10,899 start_services  281 : [*] Processing agent: apfell
INFO 2023-04-03 21:17:10,902 syncPayloadData  104 : [*] Processing command jsimport
INFO 2023-04-03 21:17:10,902 syncPayloadData  104 : [*] Processing command chrome_tabs
DEBUG 2023-04-03 21:17:10,915 SendRPCMessage  132 : Sending RPC message to pt_sync
INFO 2023-04-03 21:17:10,915 GetConnection  84  : [*] Trying to connect to rabbitmq at: 127.0.0.1:5672
INFO 2023-04-03 21:17:10,999 GetConnection  98  : [+] Successfully connected to rabbitmq
INFO 2023-04-03 21:17:11,038 ReceiveFromMythicDirectTopicExchange  306 : [*] started listening for messages on emit_webhook.new_callback
INFO 2023-04-03 21:17:11,038 ReceiveFromMythicDirectTopicExchange  306 : [*] started listening for messages on emit_webhook.new_feedback
INFO 2023-04-03 21:17:11,051 ReceiveFromMythicDirectTopicExchange  306 : [*] started listening for messages on emit_webhook.new_startup
INFO 2023-04-03 21:17:13,240 syncPayloadData  123 : [+] Successfully synced apfell

```

{% hint style="warning" %}
If you mythic instance has a randomized password for `rabbitmq_password`, then you need to make sure that the password from `Mythic/.env` after you start Mythic for the first time is copied over to your vm. You can either add this to your `rabbitmq_config.json` file or set it as an environment variable (`MYTHIC_RABBITMQ_PASSWORD`).
{% endhint %}

#### Caveats

There are a few caveats to this process over using the normal process. You're now responsible for making sure that the right python version and dependencies are installed, and you're now responsible for making sure that the user context everything is running from has the proper permissions.

One big caveat people tend to forget about is paths. Normal containers run on \*nix, but you might be doing this dev on Windows. So if you develop everything for windows paths hard-coded and then want to convert it to a normal Docker container later, that might come back to haunt you.

### Debugging Locally

Whether you're using a Docker container or not, you can load up the code in your `agent_code` folder in any IDE you want. When an agent is installed via `mythic-cli`, the entire agent folder (`agent_code` and `mythic`) is mapped into the Docker container. This means that any edits you make to the code is automatically reflected inside of the container without having to restart it (pretty handy). The only caveat here is if you make modifications to the python or golang definition files will require you to restart your container to load up the changes `sudo ./mythic-cli start [payload name]`. If you're making changes to those from a non-Docker instance, simply stop your `python3.8 main.py` and start it again. This effectively forces those files to be loaded up again and re-synced over to Mythic.

#### Debugging Agent Code Locally

If you're doing anything more than a typo fix, you're going to want to test the fixes/updates you've made to your code before you bother uploading it to a GitHub project, re-installing it, creating new agents, etc. Luckily, this can be super easy.

Say you have a Visual Studio project set up in your `agent_code` directory and you want to just "run" the project, complete with breakpoints and configurations so you can test. The only problem is that your local build needs to be known by Mythic in some way so that the Mythic UI can look up information about your agent, your "installed" commands, your encryption keys, etc.&#x20;

To do this, you first need to generate a payload in the Mythic UI (or via Mythic's Scripting). You'll select any C2 configuration information you need, any commands you want baked in, etc. When you click to build, all of that configuration will get sent to your payload type's "build" function in `mythic/agent_functions/builder.py`. Even if you don't have your container running or it fails to build, no worries, Mythic will first save everything off into the database before trying to actually build the agent. In the Mythic UI, now go to your payloads page and look for the payload you just tried to build. Click to view the information about the payload and you'll see a summary of all the components you selected during the build process, along with some additional pieces of information (payload UUID and generated encryption keys).&#x20;

Take that payload UUID and the rest of the configuration and stamp it into your `agent_code` build. For some agents this is as easy as modifying the values in a Makefile, for some agents this can all be set in a `config` file of some sort, but however you want to specify this information is up to you. Once all of that is set, you're free to run your agent from within your IDE of choice and you should see a callback in Mythic. At this point, you can do whatever code mods you need, re-run your code, etc.&#x20;

#### Callbacks Aplenty

Following from the previous section, if you just use the payload UUID and run your agent, you _should_ end up with a new callback each time. That can be ideal in some scenarios, but sometimes you're doing quick fixes and want to just keep tasking the same callback over and over again. To do this, simply pull the callback UUID and encryption keys from the callback information on the active callbacks page and plug that into your agent. Again, based on your agent's configuration, that could be as easy as modifying a Makefile, updating a config file, or you might have to manually comment/uncomment some lines of code. Once you're reporting back with the callback UUID instead of the payload UUID and using the right encryption keys, you can keep re-running your build without creating new callbacks each time.
