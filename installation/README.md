# Installation

## Get the code

Pull the code from the official GitHub repository:

```bash
$ git clone https://github.com/its-a-feature/Mythic
```

{% hint style="info" %}
This is made to work with docker and docker-compose, so they both need to be installed. If docker is not installed on your ubuntu machine, you can use the `./install_docker_ubuntu.sh` script to install it for you.\
If you're running on debian, use the `./install_docker_debian.sh` instead.
{% endhint %}

{% hint style="warning" %}
You need to have Docker server version `20.10.22` or above (latest version is `23.0.1`) for Mythic and the docker containers to work properly. If you do `sudo apt upgrade` and `sudo apt install docker-compose-plugin` on a new version of Ubuntu or Debian, then you should be good. You can check your version with `sudo docker version`.
{% endhint %}

{% hint style="danger" %}
Mythic is normally installed on Linux. If you want to install on macOS, you need to use orbstack (not Docker Desktop). This because macOS's Docker Desktop doesn't support host networking, which the C2 containers need to dynamically open up ports.
{% endhint %}

{% hint style="info" %}
It's recommended to run Mythic on a VM with at least 2CPU and 4GB Ram.
{% endhint %}

### Make the mythic-cli

All configuration is done via the `mythic-cli` binary. However, to help with GitHub sizes, the `mythic-cli` binary is no longer distributed with the main Mythic repository. Instead, you will need to make the binary via `sudo make` from the main Mythic folder. This will create the build container for the mythic-cli, build the binary, and copy it into your main Mythic folder automatically. From there on, you can use the `mythic-cli` binary like normal.

### Configure your installation

Mythic configuration is all done via `Mythic/.env`, which means for your configuration you can either add/edit values there or add them to your environment.

{% hint style="info" %}
Mythic/.env doesn't exist by default. You can either let Mythic create it for you when you run `sudo ./mythic-cli start` for the first time or you can create it ahead of time with just the variables you want to configure.
{% endhint %}

If you need to run `mythic-cli` as root for Docker and you set your environment variables as a user, be sure to run `sudo -E ./mythic-cli` so that your environment variables are carried over into your sudo call. The following are the default values that Mythic will generate on first execution of `sudo ./mythic-cli start` unless overridden:

```python
ALLOWED_IP_BLOCKS="0.0.0.0/0,::/0"
COMPOSE_PROJECT_NAME="mythic"
DEBUG_LEVEL="warning"
DEFAULT_OPERATION_NAME="Operation Chimera"
DEFAULT_OPERATION_WEBHOOK_CHANNEL=
DEFAULT_OPERATION_WEBHOOK_URL=
DOCUMENTATION_BIND_LOCALHOST_ONLY="true"
DOCUMENTATION_HOST="mythic_documentation"
DOCUMENTATION_PORT="8090"
HASURA_BIND_LOCALHOST_ONLY="true"
HASURA_CPUS="2"
HASURA_EXPERIMENTAL_FEATURES="streaming_subscriptions"
HASURA_HOST="mythic_graphql"
HASURA_MEM_LIMIT="2gb"
HASURA_PORT="8080"
HASURA_SECRET="random_password"
INSTALLED_SERVICE_CPUS="1"
INSTALLED_SERVICE_MEM_LIMIT=
JUPYTER_BIND_LOCALHOST_ONLY="true"
JUPYTER_CPUS="2"
JUPYTER_HOST="mythic_jupyter"
JUPYTER_MEM_LIMIT=
JUPYTER_PORT="8888"
JUPYTER_TOKEN="mythic"
JWT_SECRET="random_password"
MYTHIC_ADMIN_PASSWORD="random_password"
MYTHIC_ADMIN_USER="mythic_admin"
MYTHIC_API_KEY=
MYTHIC_DEBUG_AGENT_MESSAGE="false"
MYTHIC_REACT_BIND_LOCALHOST_ONLY="true"
MYTHIC_REACT_DEBUG="false"
MYTHIC_REACT_HOST="mythic_react"
MYTHIC_REACT_PORT="3000"
MYTHIC_SERVER_BIND_LOCALHOST_ONLY="false"
MYTHIC_SERVER_COMMAND=
MYTHIC_SERVER_CPUS="2"
MYTHIC_SERVER_DYNAMIC_PORTS="7000-7010"
MYTHIC_SERVER_DYNAMIC_PORTS_BIND_LOCALHOST_ONLY="false"
MYTHIC_SERVER_GRPC_PORT="17444"
MYTHIC_SERVER_HOST="mythic_server"
MYTHIC_SERVER_MEM_LIMIT=
MYTHIC_SERVER_PORT="17443"
MYTHIC_SYNC_CPUS="2"
MYTHIC_SYNC_MEM_LIMIT=
NGINX_BIND_LOCALHOST_ONLY="false"
NGINX_HOST="mythic_nginx"
NGINX_PORT="7443"
NGINX_USE_IPV4="true"
NGINX_USE_IPV6="true"
NGINX_USE_SSL="true"
POSTGRES_BIND_LOCALHOST_ONLY="false"
POSTGRES_CPUS="2"
POSTGRES_DB="mythic_db"
POSTGRES_DEBUG="false"
POSTGRES_HOST="mythic_postgres"
POSTGRES_MEM_LIMIT=
POSTGRES_PASSWORD="random_password"
POSTGRES_PORT="5432"
POSTGRES_USER="mythic_user"
RABBITMQ_BIND_LOCALHOST_ONLY="false"
RABBITMQ_CPUS="2"
RABBITMQ_HOST="mythic_rabbitmq"
RABBITMQ_MEM_LIMIT=
RABBITMQ_PASSWORD="random_password"
RABBITMQ_PORT="5672"
RABBITMQ_USER="mythic_user"
RABBITMQ_VHOST="mythic_vhost"
REBUILD_ON_START="false"
WEBHOOK_DEFAULT_ALERT_CHANNEL=
WEBHOOK_DEFAULT_CALLBACK_CHANNEL=
WEBHOOK_DEFAULT_CUSTOM_CHANNEL=
WEBHOOK_DEFAULT_FEEDBACK_CHANNEL=
WEBHOOK_DEFAULT_STARTUP_CHANNEL=
WEBHOOK_DEFAULT_URL=

```

A few important notes here:

* `MYTHIC_SERVER_PORT` will be the port opened on the server where you're running Mythic. The `NGINX_PORT` is the one that's opened by Nginx and acts as a reverse proxy to all other services. The `NGINX_PORT` is the one you'll connect to for your web user interface and should be the only port you need to expose externally (unless you prefer to SSH port forward your web UI port).
* The `allowed_ip_blocks` allow you to restrict access to the `login` page of Mythic. This should be set as a series of netblocks with **NO** host bits set - i.e. `127.0.0.0/16,192.168.10.0/24,10.0.0.0/8`
* `*_BIND_LOCALHOST_ONLY` - these settings determine if the associated container binds the port to `127.0.0.1:port` or `0.0.0.0:port`. These are all set to true (except for the nginx container) by default so that you're not exposing these services externally.

{% hint style="warning" %}
If you want to have a services (agent, c2 profile, etc) on a host _other_ than where the Mythic server is running, then you need to make sure that RABBITMQ\_BIND\_LOCALHOST\_ONLY and MYTHIC\_SERVER\_BIND\_LOCALHOST\_ONLY are both set to `false` so that your remote services can access Mythic.
{% endhint %}

{% hint style="warning" %}
The above configuration does _NOT_ affect the port or SSL information related to your agents or callback information. It's strictly for your operator web UI.
{% endhint %}

When the `mythic_server` container starts for the first time, it goes through an initialization step where it uses the password and username from `Mythic/.env` to create the `mythic_admin_user` user. Once the database exists, the `mythic_server` container no longer uses that value.

### mythic-cli

The `mythic-cli` binary is used to start/stop/configure/install components of Mythic. You can see the help menu at any time with `mythic-cli -h`, `mythic-cli --help` or `mythic-cli help`.

```
Mythic CLI is a command line interface for managing the Mythic
application and associated containers and services. Commands are grouped by their use.

Usage:
  mythic-cli [command]

Available Commands:
  add         Add local service folder to docker compose
  build       Build/rebuild a specific container
  config      Display or adjust the configuration
  database    interact with the database
  help        Help about any command
  install     install services from GitHub or local folders
  logs        Get docker logs from a running service
  mythic_sync work to mythic_sync install/uninstall
  rabbitmq    interact with the rabbitmq
  remove      Remove local service folder from docker compose
  restart     Start all of Mythic
  start       Start Mythic containers
  status      Get current Mythic container status
  stop        Stop all of Mythic
  test        test mythic service connections
  uninstall   uninstall services locally and remove them from disk

Flags:
  -h, --help   help for mythic-cli

Use "mythic-cli [command] --help" for more information about a command.

```

### Installing Agents / C2 Profiles

By default, Mythic does not come with any Payload Types (agents) or C2 Profiles. This is for a variety of reasons, but one of the big ones being time/space requirements - all Payload Types and C2 Profiles have their own Docker containers, and as such, collectively they could eat up a lot of space on disk. Additionally, having them split out into separate repositories makes it much easier to keep them updated.

Available Mythic Agents can be found on GitHub at [https://github.com/MythicAgents](https://github.com/MythicAgents)

Available Mythic C2 Profiles can be found on GitHub at [https://github.com/MythicC2Profiles](https://github.com/MythicC2Profiles)

To install a Payload Type or C2 Profile, use the `mythic-cli` binary with:

```
sudo ./mythic-cli install github <url>
```

If you have an agent already installed, but want to update it, you can do the same command again. If you supply a `-f` at the end, then Mythic will automatically overwrite the current version that's installed, otherwise you'll be prompted for each piece.

{% hint style="info" %}
You won't be able to create any payloads within Mythic until you have at least one Agent and a matching C2 Profile installed
{% endhint %}

### Logging

If you're wanting to enable SIEM-based logging, install the `basic_logger` via the mythic cli `sudo ./mythic-cli install github https://github.com/MythicC2Profiles/basic_logger`. This profile listens to the `emit_log` RabbitMQ queue and allows you to configure how you want to save/modify the logs. By default they just go to stdout, but you can configure it to write out to files or even submit the events to your own SIEM.

```
file_upload (file staged on mythic as part of tasking with the intent to get sent to the agent)
file_manual_upload (file staged on mythic as part of a user manually hosting it)
file_screenshot (file is a screenshot from the agent)
file_download (file is downloaded from agent to mythic)
artifact_new (new artifact created - think IOC)
eventlog_new (new eventlog message)
eventlog_modified (eventlog was modified, like resolving an issue or changing their message)
payload_new (new payload created)
task_mitre_attack (a task was associated with a new mitre attack technique)
task_new (a new task was created)
task_completed (a task completed)
task_comment (somebody added/removed/edited a comment on a task)
credential_new (a new credential was added to the store)
credential_modified (a credential was modified)
response_new (a new response for the user to see)
keylog_new (a new keylog entry)
callback_new (new callback registered)
```

### Start Mythic

If you came here right from the previous section, your Mythic instance should already be up and running. Check out the next section to confirm that's the case. If at any time you wish to stop Mythic, simply run `sudo ./mythic-cli stop` and if you want to start it again run `sudo ./mythic-cli start`. If Mythic is currently running and you need to make a change, you can run `sudo ./mythic-cli restart` again without any issue, that command will automatically stop things and then restart them.

The default username is `mythic_admin`, but that user's password is randomly generated when Mythic is started for the first time. You can find this random value in the `Mythic/.env` file. Once Mythic has started at least once, this value is no longer needed, so you can edit or remove this entry from the `Mythic/.env` file.

{% hint style="info" %}
Mythic starts with NO C2 Profiles or Agents pre-installed. Due to size issues and the growing number of agents, this isn't feasible. Instead. use the `./mythic-cli install github <url> [branch] [-f]` command to install an agent from a GitHub (or GitLab) repository.
{% endhint %}

### Troubleshooting installation and connection

If something seems off, here's a few places to check:

* Run `sudo ./mythic-cli status` to give a status update on all of the docker containers. They should all be up and running. If one is exited or has only been up for less than 30 seconds, that container might be your issue. All of the Mythic services will also report back a health check which can be useful to determine if a certain container is having issues. The status command gives a lot of information about what services are running, on which ports, and if they're externally accessible or not.

```
MYTHIC SERVICE		WEB ADDRESS		BOUND LOCALLY
Nginx (Mythic Web UI)	https://127.0.0.1:7443	 false
Mythic Backend Server	http://127.0.0.1:17443	 false
Hasura GraphQL Console	http://127.0.0.1:8080	 true
Jupyter Console		http://127.0.0.1:8888	 true
Internal Documentation	http://127.0.0.1:8090	 true
								
ADDITIONAL SERVICES	IP			PORT	BOUND LOCALLY
Postgres Database	127.0.0.1		5432	 false
React Server		192.168.53.152		3000	 true
RabbitMQ		127.0.0.1		5672	 false
								
				
Mythic Main Services
CONTAINER NAME		STATE		STATUS					PORTS
mythic_documentation	running		Up 38 seconds (healthy)			8090/tcp -> 127.0.0.1:8090
mythic_graphql		running		Up 36 seconds (healthy)			8080/tcp -> 127.0.0.1:8080
mythic_jupyter		running		Up 41 seconds (healthy)			8888/tcp -> 127.0.0.1:8888
mythic_nginx		running		Up 35 seconds (healthy)			7443/tcp -> :::7443, 7443
mythic_postgres		running		Up 39 seconds (healthy)			5432/tcp -> :::5432, 5432
mythic_rabbitmq		running		Up 40 seconds (health: starting)	5672/tcp -> :::5672, 5672
mythic_server		running		Up 37 seconds (health: starting)	7000/tcp -> :::7000, 7001/tcp -> :::7001, 7002/tcp -> :::7002, 7003/tcp -> :::7003, 7004/tcp -> :::7004, 7005/tcp -> :::7005, 7006/tcp -> :::7006, 7007/tcp -> :::7007, 7008/tcp -> :::7008, 7009/tcp -> :::7009, 7010/tcp -> :::7010, 17443/tcp -> :::17443, 17444/tcp -> :::17444, 7000, 7001, 7002, 7003, 7004, 7005, 7006, 7007, 7008, 7009, 7010, 17443, 17444
											
Installed Services
CONTAINER NAME		STATE		STATUS		PORTS
no_translator		running		Up 43 seconds	
service_wrapper		running		Up 42 seconds	
```

* To check the logs of any container, run `sudo ./mythic-cli logs [container_name]`. For example, to see the output of mythic\_server, run `sudo ./mythic-cli logs mythic_server`. This will help track down if the last thing that happened was an error of some kind.
* If all of that looks ok, but something still seems off, it's time to check the browser.
  * First open up the developer tools for your browser and see if there are any errors that might indicate what's wrong. If there's no error though, check the network tab to see if there are any 404 errors.
  * If that's not the case, make sure you've selected a current operation (more on this in the Quick Usage section). Mythic uses websockets that pull information about your current operation to provide data. If you're not currently in an active operation (indicated at the top of your screen in big letters), then Mythic cannot provide you any data.

## Container Sizes

Mythic starts every service (web server, database, each payload type, each C2 profile, rabbitmq, documentation) in its own Docker container. As much as possible, these containers leverage common image bases to reduce size, but due to the nature of so many components, there's going to be a decent footprint. For consideration, here's the Docker footprint for a fresh install of Mythic:

```
its-a-feature@ubuntu:$ sudo docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          9         9         9.62GB    6.263GB (65%)
Containers      9         9         399.6kB   0B (0%)
Local Volumes   17        0         2.964MB   2.964MB (100%)
Build Cache     0         0         0B        0B

its-a-feature@ubuntu:$ sudo docker system df -v
Images space usage:

REPOSITORY             TAG       IMAGE ID       CREATED         SIZE      SHARED SIZE   UNIQUE SIZE   CONTAINERS
mythic_server          latest    b11659fb912a   4 minutes ago   6.889GB   6.256GB       632.5MB       1
no_translator          latest    b9e63f1a0097   14 hours ago    6.58GB    6.256GB       323.5MB       1
service_wrapper        latest    7c508916bc3e   14 hours ago    6.581GB   6.256GB       324.5MB       1
mythic_jupyter         latest    96255e6737c4   18 hours ago    996.6MB   0B            996.6MB       1
mythic_postgres        latest    9a351f9bc9ef   18 hours ago    243.1MB   7.05MB        236.1MB       1
mythic_documentation   latest    ed947afb8c27   18 hours ago    54.36MB   0B            54.36MB       1
mythic_graphql         latest    820890c9b0ad   2 weeks ago     621.8MB   0B            621.8MB       1
mythic_nginx           latest    da7a4011c460   3 weeks ago     40.72MB   7.05MB        33.67MB       1
mythic_rabbitmq        latest    07d9eda5cc97   19 months ago   133.3MB   0B            133.3MB       1

Containers space usage:

CONTAINER ID   IMAGE                  COMMAND                  LOCAL VOLUMES   SIZE      CREATED         STATUS                   NAMES
08e5869c4b8a   mythic_postgres        "docker-entrypoint.s…"   0               63B       4 minutes ago   Up 3 minutes (healthy)   mythic_postgres
1bc0306aa920   mythic_graphql         "docker-entrypoint.s…"   0               378kB     4 minutes ago   Up 3 minutes (healthy)   mythic_graphql
41e982d38a14   mythic_nginx           "/docker-entrypoint.…"   0               2B        4 minutes ago   Up 3 minutes (healthy)   mythic_nginx
a0f1df25e66b   mythic_documentation   "hugo server -p 8090"    0               0B        4 minutes ago   Up 3 minutes (healthy)   mythic_documentation
a98ba7b3aa64   mythic_jupyter         "tini -g -- start.sh…"   0               21.2kB    4 minutes ago   Up 4 minutes (healthy)   mythic_jupyter
9d92d8397a87   mythic_rabbitmq        "docker-entrypoint.s…"   0               756B      4 minutes ago   Up 3 minutes (healthy)   mythic_rabbitmq
871405214da0   service_wrapper        "/bin/sh -c 'make ru…"   0               0B        4 minutes ago   Up 4 minutes             service_wrapper
3f7a4f72a82d   no_translator          "/bin/sh -c 'make ru…"   0               0B        4 minutes ago   Up 4 minutes             no_translator
79295b4bc031   mythic_server          "/bin/bash -c 'cp /m…"   0               0B        4 minutes ago   Up 3 minutes (healthy)   mythic_server

Local Volumes space usage:

VOLUME NAME                                                        LINKS     SIZE
1bdf5e715217e03b9e10e33b7c7e55e1ddb8898c9da53da5fc42b72c77e05ea3   0         0B
473660a24b907ce4194806b0584c236964d2b0fe0e8690057d470eaee8fcbe97   0         0B
jupyter                                                            0         2.911MB
12f257df33085912785808aa5b1e4c29910ae4ddcad498b933763668263f770d   0         93B
780ee9b1d726ac35d5ef0d2ff19f155d0e2bff437f522d6fbe984499a27c6202   0         0B
9de90d6f8308c2cc9c19043373b4f9f45d000b7ecea8b3682df224e8f9a79ada   0         0B
d0776a4a8e535a9d35f444413be020d51f0cdfd473c773c5127b5d31d95bdb61   0         0B
dbcfcf81791e768acb83c811de9d3e57172c727f83f92f8b4d7f6de4a39e84d1   0         52.77kB
documentation-docker                                               0         166B
0cd941f7e6b4b25ab1e13f1d575f2f1caaacb4664eaa9d6c404ad2347554a7fb   0         0B
cefb153314333ecb272945798c1225a5545159ae41aa92aeaa912fe017882ba8   0         93B
9c870d0f01bcbc303eb6fbf4a5a1e1bdd2c70854c02c737bb0542c5f56c0c040   0         0B
cfaacc90b7b34bdce1ed35481cab48540e6f58fd948b4bc59c5a18ecd87e00c9   0         0B
fd69883c79fe52f5108558f84b8989943b47406ac96de68c555ac7cbebb1a66b   0         0B
fe82f2f0c818cbe678db9015aebaae811fcb88e55c5039d3b8d8e82034c1b18a   0         93B
mythic                                                             0         88B
1c181b8dbadf04787e4e3faeab9b7a863d3c78a462068e9165ce4da258c31564   0         0B

Build cache usage: 0B

```

If you want to save space or if you know you're not going to be using a specific container, you can remove that container from docker-compose with `sudo ./mythic-cli remove [container name]`
