# Installation

## Get the code

Pull the code from the official GitHub repository:

```bash
$ git clone https://github.com/its-a-feature/Mythic
```

{% hint style="info" %}
&#x20;This is made to work with docker and docker-compose, so they both need to be installed. If docker is not installed on your ubuntu machine, you can use the `./install_docker_ubuntu.sh` script to install it for you. \
If you're running on debian, use the `./install_docker_debian.sh` instead.
{% endhint %}

{% hint style="danger" %}
Mythic must be installed on Linux. While macOS supports Docker and Docker-Compose, macOS doesn't handle the shared host networking that Mythic relies on. You can still access the Browser interface from any OS, but the Mythic instance must be installed on Linux
{% endhint %}

### Configure your installation

Mythic configuration is all done via `Mythic/.env`, which means for your configuration you can either add/edit values there or add them to your environment.&#x20;

{% hint style="info" %}
Mythic/.env doesn't exist by default. You can either let Mythic create it for you when you run `sudo ./mythic-cli mythic start` for the first time or you can create it ahead of time with just the variables you want to configure
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

* &#x20;`MYTHIC_SERVER_PORT`  will be the port opened on the server where you're running Mythic. The `NGINX_PORT` is the one that's opened by Nginx and acts as a reverse proxy to all other services. The `NGINX_PORT` is the one you'll connect to for your web user interface and should be the only port you need to expose externally (unless you prefer to SSH port forward your web UI port).&#x20;
* &#x20;The `allowed_ip_blocks` allow you to restrict access to the `login` page of Mythic. This should be set as a series of netblocks with **NO** host bits set - i.e. `127.0.0.0/16,192.168.10.0/24,10.0.0.0/8`&#x20;
* `excluded_c2_profiles` and `excluded_payload_types` allows you to exclude certain docker containers from starting. This is helpful if you know you're not going to use certain payload types or c2 profiles and want to cut down on time/space requirements. These are both just a comma-separated series of agent/c2 names like: `apfell,atlas` or `http,dynamichttp`.

{% hint style="warning" %}
The above configuration does _NOT_ affect the port or SSL information related to your agents or callback information. It's strictly for your operator web UI.
{% endhint %}

When the `mythic_server` container starts for the first time, it goes through an initialization step where it uses the password and username from `Mythic/.env` to create the `mythic_admin_user` user. Once the database exists, the `mythic_server` container no longer uses that value.

You'll notice that a lot of the containers have a `*_BIND_LOCALHOST_ONLY` environment variable associated with them. To help prevent Mythic from opening up port unnecessarily, Mythic binds as many containers as possible to the `mythic_default` Docker network and binds exposed ports to `127.0.0.1`. If you set the `*_BINDLOCALHOSTONLY` environment variable to `false` then Mythic will bind that port to `0.0.0.0` instead. The only container that isn't bound to localhost by default is `nginx` so that you can access that web user interface without having to do a port forward to reach it.

### mythic-cli

The `mythic-cli` binary is used to start/stop/configure/install components of Mythic. You can see the help menu at any time with `mythic-cli -h`, `mythic-cli --help` or `mythic-cli help`.&#x20;

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

By default, Mythic does not come with any Payload Types (agents) or C2 Profiles. This is for a variety of reasons, but one of the big ones being time/space requirements - all Payload Types and C2 Profiles have their own Docker containers, and as such, collectively they could eat up a lot of space on disk. Additionally, having them split out into separate repositories makes it much easier to keep them updated.&#x20;

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

The `web_log_size` and `web_keep_logs` refers only to keeping web logs (i.e. web traffic hitting Mythic). If you're wanting to enable SIEM-based logging, set the `siem_log_name` to anything but an empty string. Mythic will create that file if it doesn't exist, and log to that file. The following things are logged currently:

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

If you came here right from the previous section, your Mythic instance should already be up and running. Check out the next section to confirm that's the case. If at any time you wish to stop Mythic, simply run `sudo ./mythic-cli mythic stop` and if you want to start it again run `sudo ./mythic-cli mythic start`. If Mythic is currently running and you need to make a change, you can run `sudo ./mythic-cli mythic start` again without any issue, that command will automatically stop things and then restart them.

The default username is `mythic_admin`, but that user's password is randomly generated when Mythic is started for the first time. You can find this random value in the `Mythic/.env` file. Once Mythic has started at least once, this value is no longer needed, so you can edit or remove this entry from the `Mythic/.env` file.&#x20;

{% hint style="info" %}
Mythic starts with NO C2 Profiles or Agents pre-installed. Due to size issues and the growing number of agents, this isn't feasible. Instead. use the `./mythic-cli install github <url> [branch] [-f]` command to install an agent from a GitHub (or GitLab) repository.&#x20;
{% endhint %}

### Troubleshooting installation and connection

If something seems off, here's a few places to check:

* Run `sudo ./mythic-cli status` to give a status update on all of the docker containers. They should all be up and running. If one is exited or has only been up for less than 30 seconds, that container might be your issue.
  * Your output will be similar to the following. Notice how the `mythic_server` docker container shows a status of `Exited`? That looks like an issue

```
Core mythic services:  mythic_server, mythic_postgres, mythic_rabbitmq
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
544a64a93894        mythic_rabbitmq     "/init.sh"               3 days ago          Up 3 days                               mythic_rabbitmq
ed9d07847839        mythic_postgres     "docker-entrypoint.s…"   3 days ago          Up 3 days                               mythic_postgres
a79cec76ee88        mythic_server       "./wait-for-postgres…"   4 weeks ago         Exited (1) 30 (seconds) ago                mythic_server

C2_Profile endpoints
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
d19a3424fc39        http                "/Mythic_service/c2_…"   3 days ago          Up 3 days                               http
bb8cca7066d7        dynamichttp         "/Mythic_service/c2_…"   3 days ago          Up 3 days                               dynamichttp
0ada364906dc        chrome-server       "/Mythic_service/c2_…"   3 days ago          Up 3 days                               chrome-server

Payload Type Endpoints
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
5d9c4f84eb7d        poseidon            "/Mythic_service/pay…"   3 days ago          Up 3 days                               poseidon
587bc657206f        serviceexe          "/Mythic_service/pay…"   3 days ago          Up 3 days                               serviceexe
```

* To check the logs of any container, run `sudo ./mythic-cli logs [container_name]`. For example, to see the output of our stopped container, run `sudo ./mythic-cli logs mythic_server`. This will help track down if the last thing that happened was an error of some kind.
* If all of that looks ok, but something still seems off, it's time to check the browser.
  * If you're seeing "Session Expired, Please Refresh", "Socket errored, please refresh", or "Socket closed, please refresh", then there's an issue with your websocket connections.&#x20;
  * First open up the developer tools for your browser and see if there are any errors that might indicate what's wrong. If there's no error though, check the network tab to see if there are any 404 errors.
  * If that's not the case, make sure you've selected a current operation (more on this in the Quick Usage section). Mythic uses websockets that pull information about your current operation to provide data. If you're not currently in an active operation (indicated at the top of your screen in big letters), then Mythic cannot provide you any data.

## Container Sizes

Mythic starts every service (web server, database, each payload type, each C2 profile, rabbitmq, documentation) in its own Docker container. As much as possible, these containers leverage common image bases to reduce size, but due to the nature of so many components, there's going to be a decent footprint. For consideration, here's the Docker footprint for a fresh install of Mythic:

```
its-a-feature@ubuntu:$ sudo docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              15                  10                  8.274GB             6.266GB (75%)
Containers          12                  12                  2.992MB             0B (0%)
Local Volumes       1                   0                   166B                166B (100%)
Build Cache         0                   0                   0B                  0B
its-a-feature@ubuntu:$ sudo docker system df -v
Images space usage:

REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE                SHARED SIZE         UNIQUE SIZE         CONTAINERS
mythic_documentation                   latest              297df1e2f151        12 minutes ago      48.01MB             48MB                166B                1
poseidon                               latest              2ba62d144c91        12 minutes ago      3.508GB             3.482GB             26.85MB             1
leviathan                              latest              9ca4e9803436        14 minutes ago      1.021GB             922.7MB             98.47MB             1
dynamichttp                            latest              7a17ef432456        16 minutes ago      738.7MB             711.2MB             27.43MB             2
apfell_mythic                          latest              1c7d1ce07f67        16 minutes ago      939.2MB             702.7MB             236.5MB             1
apfell_rabbitmq                        latest              f47d738256c3        17 minutes ago      148.8MB             148.8MB             2.414kB             1
apfell_postgres                        latest              66c45a1c2200        17 minutes ago      250.7MB             250.7MB             20.88kB             1
itsafeaturemythic/xgolang_payload      0.0.5               5ae6a5e9c5e5        8 days ago          3.482GB             3.482GB             0B                  0
klakegg/hugo                           latest              1af18a30400a        3 weeks ago         48MB                48MB                0B                  0
itsafeaturemythic/python36_c2profile   0.0.2               bb950bf20d86        8 weeks ago         711.2MB             711.2MB             0B                  1
itsafeaturemythic/python36_payload     0.0.5               3507a2c336a7        2 months ago        922.7MB             922.7MB             0B                  1
itsafeaturemythic/csharp_payload       0.0.5               4345127d1907        2 months ago        1.619GB             0B                  1.619GB             2
postgres                               9.4                 ed5a45034282        6 months ago        250.7MB             250.7MB             0B                  0
python                                 3.6-jessie          890456b21ed5        13 months ago       702.7MB             702.7MB             0B                  0
rabbitmq                               3.7.6-management    500d74765467        2 years ago         148.8MB             148.8MB             0B                  0

Containers space usage:

CONTAINER ID        IMAGE                  COMMAND                  LOCAL VOLUMES       SIZE                CREATED             STATUS              NAMES
50b9b40d826d        mythic_documentation   "hugo server"            0                   0B                  12 minutes ago      Up 12 minutes       documentation
27c9c16a2920        servicewrapper         "/Mythic_service/pay…"   0                   0B                  12 minutes ago      Up 12 minutes       servicewrapper
4bffb5f83a08        poseidon               "/Mythic_service/pay…"   0                   0B                  12 minutes ago      Up 12 minutes       poseidon
d8841127ed15        leviathan              "/Mythic_service/pay…"   0                   478kB               14 minutes ago      Up 14 minutes       leviathan
f4f1d6a2c1fb        atlas                  "/Mythic_service/pay…"   0                   0B                  14 minutes ago      Up 14 minutes       atlas
e3363eec0c23        apfell                 "/Mythic_service/pay…"   0                   478kB               15 minutes ago      Up 15 minutes       apfell
a66f3c5acf31        websocket              "/Mythic_service/c2_…"   0                   505kB               16 minutes ago      Up 16 minutes       websocket
e340ed52df59        http                   "/Mythic_service/c2_…"   0                   533kB               16 minutes ago      Up 16 minutes       http
aa73e22acf9b        dynamichttp            "/Mythic_service/c2_…"   0                   505kB               16 minutes ago      Up 16 minutes       dynamichttp
6ce00c1e4811        apfell_mythic          "./wait-for-postgres…"   0                   493kB               16 minutes ago      Up 16 minutes       mythic_server
1c6416ebfeae        apfell_postgres        "docker-entrypoint.s…"   0                   63B                 16 minutes ago      Up 16 minutes       mythic_postgres
2c6f41370dd9        apfell_rabbitmq        "/init.sh"               0                   144B                16 minutes ago      Up 16 minutes       mythic_rabbitmq

Local Volumes space usage:

VOLUME NAME            LINKS               SIZE
documentation-docker   0                   166B

Build cache usage: 0B

CACHE ID            CACHE TYPE          SIZE                CREATED             LAST USED           USAGE               SHARED
```

If you want to save space or if you know you're not going to be using a specific container, add that C2 profile or Payload Type name to the appropriate `exclude` list in the `config.json` specified above. That indicates to Mythic to not even build or start that container.
