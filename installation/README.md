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
DEBUG_LEVEL="debug"
DEFAULT_OPERATION_NAME="Operation Chimera"
DEFAULT_OPERATION_WEBHOOK_CHANNEL=
DEFAULT_OPERATION_WEBHOOK_URL=
DOCUMENTATION_BIND_LOCALHOST_ONLY="true"
DOCUMENTATION_HOST="mythic_documentation"
DOCUMENTATION_PORT="8090"
DOCUMENTATION_USE_BUILD_CONTEXT="false"
DOCUMENTATION_USE_VOLUME="true"
GLOBAL_DOCKER_LATEST="v0.0.3"
GLOBAL_MANAGER="docker"
GLOBAL_SERVER_NAME="mythic"
HASURA_BIND_LOCALHOST_ONLY="true"
HASURA_CPUS="2"
HASURA_EXPERIMENTAL_FEATURES="streaming_subscriptions"
HASURA_HOST="mythic_graphql"
HASURA_MEM_LIMIT="2gb"
HASURA_PORT="8080"
HASURA_SECRET="random password"
HASURA_USE_BUILD_CONTEXT="false"
HASURA_USE_VOLUME="true"
INSTALLED_SERVICE_CPUS="1"
INSTALLED_SERVICE_MEM_LIMIT=
JUPYTER_BIND_LOCALHOST_ONLY="true"
JUPYTER_CPUS="2"
JUPYTER_HOST="mythic_jupyter"
JUPYTER_MEM_LIMIT=
JUPYTER_PORT="8888"
JUPYTER_TOKEN="mythic"
JUPYTER_USE_BUILD_CONTEXT="false"
JUPYTER_USE_VOLUME="true"
JWT_SECRET="random password"
MYTHIC_ADMIN_PASSWORD="random password"
MYTHIC_ADMIN_USER="mythic_admin"
MYTHIC_API_KEY=
MYTHIC_DEBUG_AGENT_MESSAGE="false"
MYTHIC_REACT_BIND_LOCALHOST_ONLY="true"
MYTHIC_REACT_DEBUG="false"
MYTHIC_REACT_HOST="mythic_react"
MYTHIC_REACT_PORT="3000"
MYTHIC_REACT_USE_BUILD_CONTEXT="false"
MYTHIC_REACT_USE_VOLUME="true"
MYTHIC_SERVER_BIND_LOCALHOST_ONLY="true"
MYTHIC_SERVER_COMMAND=
MYTHIC_SERVER_CPUS="2"
MYTHIC_SERVER_DYNAMIC_PORTS="7000-7010,1080"
MYTHIC_SERVER_DYNAMIC_PORTS_BIND_LOCALHOST_ONLY="false"
MYTHIC_SERVER_GRPC_PORT="17444"
MYTHIC_SERVER_HOST="mythic_server"
MYTHIC_SERVER_MEM_LIMIT=
MYTHIC_SERVER_PORT="17443"
MYTHIC_SERVER_USE_BUILD_CONTEXT="false"
MYTHIC_SERVER_USE_VOLUME="true"
MYTHIC_SYNC_CPUS="2"
MYTHIC_SYNC_MEM_LIMIT=
NGINX_BIND_LOCALHOST_ONLY="false"
NGINX_HOST="mythic_nginx"
NGINX_PORT="7443"
NGINX_USE_BUILD_CONTEXT="false"
NGINX_USE_IPV4="true"
NGINX_USE_IPV6="false"
NGINX_USE_SSL="true"
NGINX_USE_VOLUME="true"
POSTGRES_BIND_LOCALHOST_ONLY="false"
POSTGRES_CPUS="2"
POSTGRES_DB="mythic_db"
POSTGRES_DEBUG="false"
POSTGRES_HOST="mythic_postgres"
POSTGRES_MEM_LIMIT=
POSTGRES_PASSWORD="random password"
POSTGRES_PORT="5432"
POSTGRES_USE_BUILD_CONTEXT="false"
POSTGRES_USE_VOLUME="true"
POSTGRES_USER="mythic_user"
RABBITMQ_BIND_LOCALHOST_ONLY="true"
RABBITMQ_CPUS="2"
RABBITMQ_HOST="mythic_rabbitmq"
RABBITMQ_MEM_LIMIT=
RABBITMQ_PASSWORD="random password"
RABBITMQ_PORT="5672"
RABBITMQ_USE_BUILD_CONTEXT="false"
RABBITMQ_USE_VOLUME="true"
RABBITMQ_USER="mythic_user"
RABBITMQ_VHOST="mythic_vhost"
REBUILD_ON_START="true"
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

## Mythic Pre-built containers

Starting with Mythic 3.2.16, Mythic pre-builds its main service containers and hosts them on GitHub. You'll see `ghcr.io/itsafeature` in the FROM line in your Dockerfiles instead of the `itsafeaturemythic/` line which is hosted on DockerHub. When Mythic gets a new `tag`, these images are pre-built, mythic-cli is updated, and the associated `push` on GitHub is updated with the new tag version.&#x20;

When you use the new `mythic-cli` to start Mythic, the `.env` variable `GLOBAL_DOCKER_LATEST` is used to determine which version of the Docker images to use. This variable is written out and saved as part of `mythic-cli` itself, so make sure when you do a `git pull` that you always run `sudo make` to get the latest `mythic-cli` as well.

As part of this, there are two new variables for each container:

* `*_USE_BUILD_CONTEXT` - This variable changes the `docker-compose` file to either set a `build-context` to read the local Dockerfile when building or to _not_ use the local build context and just set the image to use to be the one hosted on GitHub. In most cases, you're fine to leave this as `false` and just use the image hosted on GitHub. If you wanted to use another image version or if you wanted add stuff to the image that gets generated for a container, you can set this to `true` and modify the Dockerfile associated with the service.
* `*_USE_VOLUME` - This variable identifies if the local file system is mounted into the image at run-time or if a custom volume is created and mounted instead. When this is set to `true`, then a custom volume is created and mounted into the container at run time so that your local filesystem isn't used. When this is `false`, then your local filesystem is mounted like normal. One reason to mount the local file system instead of using a volume is if you wanted to make changes to something on disk and have it reflected in the container. Similarly, you can set this to `false` so that your database and downloaded files are all contained within the `Mythic` folder. Setting this to `true` will mean that volumes are used, so your saved files and database are in Docker's volume directory and not locally within the Mythic folder. It's just something to consider when it comes time to save things off or if you wanted to pull the files from disk.

## Agent Pre-built containers

Mythic is pre-building its containers so that it's faster and easier to get going while still keeping all of the flexibility of Dockerimages. This cuts down on the install/build time of the containers and reduces the general size of the images due to multi-stage Docker builds.

Agents on GitHub can also do this for free. It's pretty simple (all things considered) and provides a lot of flexibility to how you build your containers. You don't need to configure any special GitHub secrets - you just need to create the necessary yaml file as part of a certain directory of your repository so that things are kicked off on push and on tag. One of these changes is an automatic update to your `config.json` so that Mythic can also track the version associated with your agent.&#x20;

Specifically, you need to create the `.github/workflows/[name].yml` file so that GitHub will be able to handle your actions. An example from the `service_wrapper` payload is shown below:

{% @github-files/github-code-block url="https://github.com/MythicAgents/service_wrapper/blob/master/.github/workflows/docker.yml" %}

&#x20;99% of this example should work for all agents and c2 profiles. Things to change:

* `RELEASE_BRANCH` - This might need to change depending on if your branch name is `master` or `main`. (line 39)
* Updating the `remote_images.service_wrapper` (line 112) to `remote_images.[your name]` so that your `config.json` is updated appropriately. If you have multiple installs (ex: a payload in the payload\_type folder and a c2 profile in the c2\_profiles folder), then you should include this action multiple times to add each entry to your `remote_images` dictionary.
* Updating the files to update (line 121) - after building and pushing your container image, you'll need to update the corresponding files with the new version. Once they're updated, you need to make sure those changes actually get saved and pushed with your tag updated to that new pushed version. This line points to which files to add to the new commit.
* There are a few places in this example that use a `working_directory` that points to where the Dockerfiles are to use for building and saving changes. Make sure those paths reflect your agent/c2 profiles paths. You can find them in this example if you search for `service_wrapper`.

In addition to this GitHub Actions file, you'll need two Dockerfiles - one that's used to pre-build your images, and one that's updated with a `FROM ghcr.io/` to point to your newly created image. The common convention used here is to create a `.docker` folder with your agent/c2 profile and the full Dockerfile in there (along with any necessary resources). Below is the example build Dockerfile in the `.docker/Dockerfile` for the `service_wrapper` payload type:

{% @github-files/github-code-block url="https://github.com/MythicAgents/service_wrapper/blob/master/Payload_Type/service_wrapper/.docker/Dockerfile" %}

There's a few things to note here:

* we're using one of the Mythic DockerHub images as a `builder` and using a different, smaller stage (`python3.11-slim-bullseye` in this case) that we copy things over to. The main thing that's going to make the images smaller is to build what's needed ahead of time and then move to a smaller image and copy things over. This is easier for C2 Profiles since you can really limit what gets moved to that second stage. This is harder for Agents though because you might still need a lot of toolchains and SDKs installed to build your agent on-demand. The best thing you can do here is to pre-pull any necessary requirements (like nuget packages or golang packages) so that at build-time for a payload you don't have to fetch them and can just build.
* Because the `service_wrapper` uses the Python `mythic-container` PyPi library instead of the Golang library, we need to make sure we have Python 3.11+ and the right PyPi packages installed. On lines 3-4 we're copying in the requirements into our `builder` container and generating `wheels` from them so that in the second stage, lines 18-19, we copy over those wheels and install those.&#x20;

Always make sure to test out your docker images to confirm you still have everything needed to build your agent. Some good reference points are any of the `.docker/Dockerfile` references in the `Mythic` repo, or the `apfell`, `service_wrapper`, `websocket`, or `http` versions.
