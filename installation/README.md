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

{% hint style="danger" %}
Mythic must be installed on Linux. While macOS supports Docker and Docker-Compose, macOS doesn't handle the shared host networking that Mythic relies on for C2 containers. You can still access the Browser interface from any OS, but the Mythic instance must be installed on Linux
{% endhint %}

### Configure your installation

Mythic configuration is all done via `Mythic/.env`, which means for your configuration you can either add/edit values there or add them to your environment.

{% hint style="info" %}
Mythic/.env doesn't exist by default. You can either let Mythic create it for you when you run `sudo ./mythic-cli start` for the first time or you can create it ahead of time with just the variables you want to configure.
{% endhint %}

If you need to run `mythic-cli` as root for Docker and you set your environment variables as a user, be sure to run `sudo -E ./mythic-cli` so that your environment variables are carried over into your sudo call. The following are the default values that Mythic will generate on first execution of `sudo ./mythic-cli mythic start` unless overridden:

```python
ALLOWED_IP_BLOCKS="0.0.0.0/0"
COMPOSE_PROJECT_NAME="mythic"
DEFAULT_OPERATION_NAME="Operation Chimera"
DOCUMENTATION_BIND_LOCALHOST_ONLY="true"
DOCUMENTATION_HOST="mythic_documentation"
DOCUMENTATION_PORT="8090"
EXCLUDED_C2_PROFILES=
EXCLUDED_PAYLOAD_TYPES=
HASURA_BIND_LOCALHOST_ONLY="true"
HASURA_HOST="mythic_graphql"
HASURA_PORT="8080"
HASURA_SECRET= random password
JWT_SECRET= random password
MYTHIC_ADMIN_PASSWORD= random password
MYTHIC_ADMIN_USER="mythic_admin"
MYTHIC_DEBUG="false"
MYTHIC_ENVIRONMENT="production"
MYTHIC_REACT_BIND_LOCALHOST_ONLY="false"
MYTHIC_REACT_HOST="mythic_react"
MYTHIC_REACT_PORT="3000"
MYTHIC_SERVER_BIND_LOCALHOST_ONLY="true"
MYTHIC_SERVER_DYNAMIC_PORTS="7000-7010"
MYTHIC_SERVER_HOST="mythic_server"
MYTHIC_SERVER_PORT="17443"
NGINX_BIND_LOCALHOST_ONLY="false"
NGINX_HOST="mythic_nginx"
NGINX_PORT="7443"
NGINX_USE_SSL="true"
POSTGRES_BIND_LOCALHOST_ONLY="true"
POSTGRES_DB="mythic_db"
POSTGRES_HOST="mythic_postgres"
POSTGRES_PASSWORD= random password
POSTGRES_PORT="5432"
POSTGRES_USER="mythic_user"
RABBITMQ_BIND_LOCALHOST_ONLY="false"
RABBITMQ_HOST="mythic_rabbitmq"
RABBITMQ_PASSWORD= random password
RABBITMQ_PORT="5672"
RABBITMQ_USER="mythic_user"
RABBITMQ_VHOST="mythic_vhost"
REBUILD_ON_START="true"
REDIS_BIND_LOCALHOST_ONLY="true"
REDIS_HOST="mythic_redis"
REDIS_PORT="6380"
SERVER_HEADER="nginx 1.2"
SIEM_LOG_NAME=
WEB_KEEP_LOGS="false"
WEB_LOG_SIZE="1024000"
```

A few important notes here:

* `MYTHIC_SERVER_PORT` will be the port opened on the server where you're running Mythic. The `NGINX_PORT` is the one that's opened by Nginx and acts as a reverse proxy to all other services. The `NGINX_PORT` is the one you'll connect to for your web user interface and should be the only port you need to expose externally (unless you prefer to SSH port forward your web UI port).
* The `allowed_ip_blocks` allow you to restrict access to the `login` page of Mythic. This should be set as a series of netblocks with **NO** host bits set - i.e. `127.0.0.0/16,192.168.10.0/24,10.0.0.0/8`
* `excluded_c2_profiles` and `excluded_payload_types` allows you to exclude certain docker containers from starting. This is helpful if you know you're not going to use certain payload types or c2 profiles and want to cut down on time/space requirements. These are both just a comma-separated series of agent/c2 names like: `apfell,atlas` or `http,dynamichttp`.

{% hint style="warning" %}
The above configuration does _NOT_ affect the port or SSL information related to your agents or callback information. It's strictly for your operator web UI.
{% endhint %}

When the `mythic_server` container starts for the first time, it goes through an initialization step where it uses the password and username from `Mythic/.env` to create the `mythic_admin_user` user. Once the database exists, the `mythic_server` container no longer uses that value.

You'll notice that a lot of the containers have a `*_BIND_LOCALHOST_ONLY` environment variable associated with them. To help prevent Mythic from opening up port unnecessarily, Mythic binds as many containers as possible to the `mythic_default` Docker network and binds exposed ports to `127.0.0.1`. If you set the `*_BINDLOCALHOSTONLY` environment variable to `false` then Mythic will bind that port to `0.0.0.0` instead. The only container that isn't bound to localhost by default is `nginx` so that you can access that web user interface without having to do a port forward to reach it.

### mythic-cli

The `mythic-cli` binary is used to start/stop/configure/install components of Mythic. You can see the help menu at any time with `mythic-cli -h`, `mythic-cli --help` or `mythic-cli help`.

```
mythic-cli usage ( v 0.0.5 ):
*************************************************************
*** source code: https://github.com/MythicMeta/Mythic_CLI ***
*************************************************************
  help
  mythic {start|stop} [service name...]
  start | restart
    Stops and Starts all of Mythic - alias for 'mythic start'
  stop
    Stop all of Mythic - alias for 'mythic stop'
  c2 {start|stop|add|remove|list} [c2profile ...]
      The add/remove subcommands adjust the docker-compose file, not manipulate files on disk
         to manipulate files on disk, use 'install' and 'uninstall' commands
  payload {start|stop|add|remove|list} [payloadtype ...]
      The add/remove subcommands adjust the docker-compose file, not manipulate files on disk
         to manipulate files on disk, use 'install' and 'uninstall' commands
  config
      *no parameters will dump the entire config*
      get [varname ...]
      set <var name> <var value>
      payload (dump out remote payload configuration variables)
      c2 (dump out remote c2 configuration variables)
  database reset
  install 
      github <url> [branch name] [-f]
      folder <path to folder> [-f]
      -f forces the removal of the currently installed version and overwrites with the new, otherwise will prompt you
         * this command will manipulate files on disk and update docker-compose
  uninstall {name1 name2 name2 ...}
      (this command removes the payload or c2 profile from disk and updates docker-compose)
  status
  logs <container name>
  mythic_sync
      install github [url] [branch name]
         * if no url is provided, https://github.com/GhostManager/mythic_sync will be used
      install folder <path to folder>
      uninstall
  version
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

If you're wanting to enable SIEM-based logging, set the `siem_log_name` to anything but an empty string. Mythic will create that file if it doesn't exist, and log to that file. The following things are logged currently:

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

Mythic does SIEM-based logging as JSON data where each entry is as follows:

`{"timestamp": "UTC Timestring", "mythic_object": "one of the values from above", "message": JSON of the actual message in question}`

To start Mythic, simply run `sudo ./mythic-cli mythic start`.

### Start Mythic

If you came here right from the previous section, your Mythic instance should already be up and running. Check out the next section to confirm that's the case. If at any time you wish to stop Mythic, simply run `sudo ./mythic-cli stop` and if you want to start it again run `sudo ./mythic-cli start`. If Mythic is currently running and you need to make a change, you can run `sudo ./mythic-cli restart` again without any issue, that command will automatically stop things and then restart them.

The default username is `mythic_admin`, but that user's password is randomly generated when Mythic is started for the first time. You can find this random value in the `Mythic/.env` file. Once Mythic has started at least once, this value is no longer needed, so you can edit or remove this entry from the `Mythic/.env` file.

{% hint style="info" %}
Mythic starts with NO C2 Profiles or Agents pre-installed. Due to size issues and the growing number of agents, this isn't feasible. Instead. use the `./mythic-cli install github <url> [branch] [-f]` command to install an agent from a GitHub (or GitLab) repository.
{% endhint %}

### Troubleshooting installation and connection

If something seems off, here's a few places to check:

* Run `sudo ./mythic-cli status` to give a status update on all of the docker containers. They should all be up and running. If one is exited or has only been up for less than 30 seconds, that container might be your issue. The status command gives a lot of information about what services are running, on which ports, and if they're externally accessible or not.
  * Your output will be similar to the following. Notice how the `mythic_server` docker container shows a status of `Exited`? That looks like an issue

```
CONTAINER NAME		MYTHIC SERVICE		WEB ADDRESS		BOUND LOCALLY
mythic_nginx		Nginx (Mythic Web UI)	https://127.0.0.1:7443	 false
mythic_server		Mythic Backend Server	http://127.0.0.1:17443	 true
mythic_graphql		Hasura GraphQL Console	http://127.0.0.1:8080	 true
mythic_documentation	Internal Documentation	http://127.0.0.1:8090	 true 

CONTAINER NAME		ADDITIONAL SERVICES	IP		PORT	BOUND LOCALLY
mythic_postgres		Postgres Database	127.0.0.1	5432	 true
mythic_redis		Redis Database		127.0.0.1	6379	 true
mythic_react		React Server		192.168.53.129	3000	 true
mythic_rabbitmq		RabbitMQ		127.0.0.1	5672	 true 

Mythic Main Services:
NAME			STATE		STATUS		PORTS
mythic_server		running		Up 15 hours	17443/tcp -> 127.0.0.1:17443, 7000, 7001, 7002, 7003, 7004, 7005, 7006, 7007, 7008, 7009, 7010
mythic_graphql		running		Up 15 hours	8080/tcp -> 127.0.0.1:8080
mythic_redis		running		Up 15 hours	6379/tcp -> 127.0.0.1:6379
mythic_rabbitmq		running		Up 15 hours	5672/tcp -> 127.0.0.1:5672
mythic_nginx		running		Up 15 hours	7443
mythic_documentation	running		Up 15 hours	8090/tcp -> 127.0.0.1:8090
mythic_postgres		running		Up 15 hours	5432/tcp -> 127.0.0.1:5432

Payload Type Services:
NAME		STATE		STATUS		PORTS
apfell		running		Up 20 Seconds	
poseidon	running		Up 15 hours	
atlas		running		Up 15 hours	

C2 Profile Services:
NAME	STATE		STATUS		PORTS
http	running		Up 15 hours	

[*] RabbitMQ is currently listening on localhost. If you have a remote PayloadType or C2Profile, they will be unable to connect
    Use 'sudo ./mythic-cli config set rabbitmq_bind_localhost_only false' and restart mythic ('sudo ./mythic-cli mythic start') to change this
[*] If you are using a remote PayloadType or C2Profile, they will need certain environment variables to properly connect to Mythic.
    Use 'sudo ./mythic-cli payload config' or 'sudo ./mythic-cli c2 config' for easy-to-use configs for these services.

```

* To check the logs of any container, run `sudo ./mythic-cli logs [container_name]`. For example, to see the output of our stopped container, run `sudo ./mythic-cli logs mythic_server`. This will help track down if the last thing that happened was an error of some kind.
* If all of that looks ok, but something still seems off, it's time to check the browser.
  * If you're seeing "Session Expired, Please Refresh", "Socket errored, please refresh", or "Socket closed, please refresh", then there's an issue with your websocket connections.
  * First open up the developer tools for your browser and see if there are any errors that might indicate what's wrong. If there's no error though, check the network tab to see if there are any 404 errors.
  * If that's not the case, make sure you've selected a current operation (more on this in the Quick Usage section). Mythic uses websockets that pull information about your current operation to provide data. If you're not currently in an active operation (indicated at the top of your screen in big letters), then Mythic cannot provide you any data.

## Container Sizes

Mythic starts every service (web server, database, each payload type, each C2 profile, rabbitmq, documentation) in its own Docker container. As much as possible, these containers leverage common image bases to reduce size, but due to the nature of so many components, there's going to be a decent footprint. For consideration, here's the Docker footprint for a fresh install of Mythic:

```
its-a-feature@ubuntu:$ sudo docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          32        32        14.8GB    6.726GB (45%)
Containers      115       11        77.25MB   75.87MB (98%)
Local Volumes   13        2         498B      254B (51%)
Build Cache     0         0         0B        0B
its-a-feature@ubuntu:$ sudo docker system df -v
Images space usage:

REPOSITORY                           TAG                IMAGE ID       CREATED         SIZE      SHARED SIZE   UNIQUE SIZE   CONTAINERS
poseidon                             latest             cdff4b42515d   15 hours ago    3.596GB   3.01GB        585.7MB       1
apfell                               latest             819f500ce428   15 hours ago    907.2MB   883.6MB       23.66MB       1
atlas                                latest             f9572f0dcc1b   5 days ago      1.756GB   1.755GB       340.7kB       1
<none>                               <none>             1d40ecbbfc93   5 days ago      1.86GB    1.755GB       105MB         0
nginx                                1.21.4-alpine      b46db85084b8   2 weeks ago     23.2MB    0B            23.2MB        1
<none>                               <none>             c0f1b5c143b6   4 weeks ago     906.7MB   883.6MB       23.1MB        0
itsafeaturemythic/csharp_payload     0.0.15             9c5714ea8fb7   2 months ago    1.755GB   1.755GB       0B            0
<none>                               <none>             b21ae223893a   2 months ago    3.874GB   3.01GB        863.9MB       0
<none>                               <none>             42aab0c8184c   2 months ago    896.2MB   883.3MB       12.92MB       0
<none>                               <none>             15156ebb5053   2 months ago    895MB     883.3MB       11.71MB       0
mythic_redis                         latest             591750eea2dc   3 months ago    32.31MB   5.595MB       26.71MB       1
mythic_postgres                      latest             fa532a7c8577   3 months ago    159.5MB   0B            159.5MB       1
mythic_documentation                 latest             22da9cd2b72a   3 months ago    52.09MB   0B            52.09MB       1
mythic_rabbitmq                      latest             4168ba9e4515   3 months ago    133.3MB   133.2MB       51.65kB       1
<none>                               <none>             6af270349f2b   3 months ago    52.91MB   5.595MB       47.31MB       0
<none>                               <none>             79749b04f4ba   3 months ago    51.61MB   42.66MB       8.951MB       0
<none>                               <none>             ac67ed6876f8   3 months ago    42.66MB   42.66MB       37B           0
<none>                               <none>             903fd92735ca   3 months ago    42.66MB   42.66MB       45B           0
<none>                               <none>             33bb9b5d0d90   3 months ago    133.2MB   133.2MB       0B            0
tcp                                  latest             b86f777d70df   4 months ago    961MB     883.3MB       77.72MB       1
<none>                               <none>             01354badf9e3   4 months ago    942.9MB   883.3MB       59.63MB       0
mythic_graphql                       latest             0d81f467f1f3   4 months ago    424.4MB   0B            424.4MB       1
<none>                               <none>             7494ffd16bba   4 months ago    112.1MB   0B            112.1MB       0
<none>                               <none>             efc50622e6f4   5 months ago    895MB     883.3MB       11.7MB        0
itsafeaturemythic/xgolang_payload    0.0.11             034e97930122   5 months ago    3.584GB   0B            3.584GB       0
<none>                               <none>             ff958fe8d254   5 months ago    883.3MB   883.3MB       220B          0
itsafeaturemythic/python38_payload   0.0.5              1f1f9958d570   5 months ago    895MB     883.3MB       11.7MB        0
<none>                               <none>             e9ef6bc79ef5   5 months ago    1.732GB   0B            1.732GB       0
<none>                               <none>             56a1d05bdbb9   6 months ago    895MB     895MB         11.58kB       0
itsafeaturemythic/python38_payload   0.0.4              b1cd2e331850   6 months ago    895MB     895MB         0B            0
mythic_server                        latest             43e62f4de2ba   7 months ago    995.6MB   883.3MB       112.3MB       1
python                               3.8.5-alpine3.12   0f03316d4a27   15 months ago   42.66MB   42.66MB       0B            0

Containers space usage:

CONTAINER ID   IMAGE                  COMMAND                  LOCAL VOLUMES   SIZE      CREATED        STATUS        NAMES
169c508098f1   apfell                 "/Mythic/mythic/payl…"   0               0B        3 hours ago    Up 3 hours    mythic_apfell_1
6dbaa8bc973e   mythic_server          "/bin/bash /Mythic/s…"   0               101kB     15 hours ago   Up 15 hours   mythic_server
1460793cf421   mythic_graphql         "docker-entrypoint.s…"   0               801kB     15 hours ago   Up 15 hours   mythic_graphql
709046591b17   mythic_redis           "docker-entrypoint.s…"   1               0B        15 hours ago   Up 15 hours   mythic_redis
0d339efc65fa   mythic_rabbitmq        "docker-entrypoint.s…"   0               768B      15 hours ago   Up 15 hours   mythic_rabbitmq
3b9547a85efa   poseidon               "/Mythic/mythic/payl…"   0               0B        15 hours ago   Up 15 hours   mythic_poseidon_1
2ad1e781c0df   atlas                  "/Mythic/mythic/payl…"   0               0B        15 hours ago   Up 15 hours   mythic_atlas_1
55e465a87b50   b86f777d70df           "/Mythic/mythic/c2_s…"   0               473kB     15 hours ago   Up 15 hours   mythic_http_1
53b26b7ef246   b46db85084b8           "/docker-entrypoint.…"   0               2B        15 hours ago   Up 15 hours   mythic_nginx
ff0c9f8b5175   mythic_documentation   "hugo server -p 8090"    0               0B        15 hours ago   Up 15 hours   mythic_documentation
fbe7cd466887   mythic_postgres        "docker-entrypoint.s…"   0               63B       15 hours ago   Up 15 hours   mythic_postgres

Local Volumes space usage:

VOLUME NAME                                                        LINKS     SIZE
fd69883c79fe52f5108558f84b8989943b47406ac96de68c555ac7cbebb1a66b   0         0B
57f0091f43fe689767fb6aba2ff22ea3299067c931142e4e7b818dcfd4e6e3b9   1         244B
0cd941f7e6b4b25ab1e13f1d575f2f1caaacb4664eaa9d6c404ad2347554a7fb   0         0B
cfaacc90b7b34bdce1ed35481cab48540e6f58fd948b4bc59c5a18ecd87e00c9   0         0B
473660a24b907ce4194806b0584c236964d2b0fe0e8690057d470eaee8fcbe97   0         0B
780ee9b1d726ac35d5ef0d2ff19f155d0e2bff437f522d6fbe984499a27c6202   0         0B
9c870d0f01bcbc303eb6fbf4a5a1e1bdd2c70854c02c737bb0542c5f56c0c040   0         0B
9de90d6f8308c2cc9c19043373b4f9f45d000b7ecea8b3682df224e8f9a79ada   0         0B
d0776a4a8e535a9d35f444413be020d51f0cdfd473c773c5127b5d31d95bdb61   0         0B
documentation-docker                                               0         166B
1bdf5e715217e03b9e10e33b7c7e55e1ddb8898c9da53da5fc42b72c77e05ea3   0         0B
1c181b8dbadf04787e4e3faeab9b7a863d3c78a462068e9165ce4da258c31564   0         0B
mythic                                                             0         88B

Build cache usage: 0B
```

If you want to save space or if you know you're not going to be using a specific container, add that C2 profile or Payload Type name to the appropriate `exclude` list in the `config.json` specified above. That indicates to Mythic to not even build or start that container.
