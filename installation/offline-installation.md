---
description: How to install Mythic and agents in an offline environment
---

# Offline Installation

{% hint style="info" %}
This guide will assume you can install Mythic on a box that has Internet access and then migrate to your offline testing/development environment.
{% endhint %}

### Setup

1. Install Mythic following the normal installation
2. With Mythic running, install any other agents or profiles you might need/want.

```
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo
```

3\. Export your docker containers. Make sure you also save the tags.

```
docker save $(docker images -q) -o mythic_images.tar
docker images | sed '1d' | awk '{print $1 " " $2 " " $3}' > mythic_tags
```

4\. Download donut from pypi. (this is apollo specific, so there might be others depending on your agent)

```
mkdir Payload_Types/apollo/depends
pip3 download donut -d Payload_Types/apollo/depends
```

Download Apollo dependencies (apollo specifically installs these dynamically within the Docker container at build-time, so pre-fetch these)

```
 wget https://www.nuget.org/api/v2/package/Fody/2.0.0 -O Payload_Types/apollo/depends/fody.2.0.0.nupkg
 wget https://www.nuget.org/api/v2/package/Costura.Fody/1.6.2 -O Payload_Types/apollo/depends/costura.fody.1.6.2.nupkg
```

5\. Tar Mythic directoy.

```
tar cfz mythic.tar.gz /Mythic
```

6\. Push `mythic_images.tar`, `mythic_tags`, and `mythic.tar.gz` to your offline box.

7\. Import docker images and restore tags.

```
docker load -i mythic_images.tar
while read REPOSITORY TAG IMAGE_ID; do echo "== Tagging $REPOSITORY $TAG $IMAGE_ID =="; docker tag "$IMAGE_ID" "$REPOSITORY:$TAG"; done < mythic_tags
```

8\. Extract Mythic directory.

```
tar xfz mythic.tar.gz
cd mythic
```

9\. Update Apollo's Dockerfile (at the time of use, it might not be 0.1.1 anymore, check [#current-payloadtype-versions](../customizing/payload-type-development/payload-type-info/container-syncing.md#current-payloadtype-versions "mention") the latest). This is apollo specific, so you might need to copy in pieces for other agents/c2 profiles depending on what components they dynamically try to install.

```
from itsafeaturemythic/csharp_payload:0.1.1

COPY ["depends/donut-0.2.2.tar.gz", "donut-0.2.2.tar.gz"]
COPY ["depends/costura.fody.1.6.2.nupkg", "costura.fody.1.6.2.nupkg"]
COPY ["depends/fody.2.0.0.nupkg", "fody.2.0.0.nupkg"]

RUN /usr/local/bin/python3.8 -m pip install /donut-0.2.2.tar.gz
RUN mkdir /mythic_nuget
RUN nuget sources add -name mythic_nuget -source /mythic_nuget
RUN nuget sources disable -name nuget.org
RUN nuget add /fody.2.0.0.nupkg -source /mythic_nuget
RUN nuget add /costura.fody.1.6.2.nupkg -source /mythic_nuget
```

10\. Start Mythic

```
sudo ./mythic-cli start
```

{% hint style="info" %}
Normally, Mythic containers will try to re-build every time you bring them down and back up. This might not be great for an offline environment. The configuration variable, `REBUILD_ON_START`, can be set to `false` to tell Mythic that the containers should specifically NOT be rebuilt when restarted.
{% endhint %}
