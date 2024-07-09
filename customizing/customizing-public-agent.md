# Customizing Public Agent

## My Changes aren't working

You installed a service into Mythic that's not yours (agent, c2, webhook, etc), made a change, but you're not seeing it? That could be from forking a public agent, making changes in your own repo and installing it with `./mythic-cli install` or just making local changes on disk. Luckily, there's a really easy solution to this.

This page walks through the various things covered in this blog post as well: [https://medium.com/@its\_a\_feature\_/agent-customization-in-mythic-tailoring-tools-for-red-team-needs-1746fd02177f](https://medium.com/@its\_a\_feature\_/agent-customization-in-mythic-tailoring-tools-for-red-team-needs-1746fd02177f)

### Remote Images

Docker containers are really amazing. They rely on "images" to create a kind of "snapshot" of a simulated VM and then turn that image into an instance of that running snapshot by creating a container. These images can do a lot of things and configure a lot of different components for you so that you can be absolutely sure that how something is set up in one environment matches another environment regardless of whatever else is installed or set up. To do this, when building the image, you identify packages to install, things to configure, build new binaries, etc.&#x20;

The downside is that creating the image in the first place can be very taxing for a CPU and for the HDD. They can be building python from source and ballooning the size of intermediary layers all over the place. To help with this, some authors of Mythic services have opted to use remote images. This means that the images are already pre-built for the general case and hosted somewhere (GitHub, DockerHub, etc).&#x20;

If you're ever curious about an agent using a remote image, you can check [https://mythicmeta.github.io/overview/](https://mythicmeta.github.io/overview/) and look at the Docker Image column. If there's something there, then the service is going to use the image hosted there by default. If you want to check locally, you'll see three new variables in your `.env` about it. For example, let's say we installed `Poseidon`:

```
POSEIDON_USE_BUILD_CONTEXT="true"
POSEIDON_USE_VOLUME="false"
POSEIDON_REMOTE_IMAGE="ghcr.io/mythicagents/poseidon:v0.0.0.14"
```

You'll see three new `.env` variables all prefixed with the name of the thing you installed.

### \*\_USE\_BUILD\_CONTEXT

The `*_USE_BUILD_CONTEXT` variable says whether or not to use the LOCAL build context to create an image or to instead use the specified `*_REMOTE_IMAGE` that's pre-built. This means that when this variable is `false` (the default), then no new local changes will be used and the pre-built image will simply be fetched and turned into a container. So, no matter how many local changes you make, you'll never see the changes.

Setting this to `true` means that the local `Dockerfile` will be used to generate the image you use for your container. It's _most likely_ the case that this Dockerfile is set up to pull in your local changes when creating the image, rebuilding things as necessary. If it's not though, then your local Dockerfile will be used to generate a new local image, but it doesn't _guarantee_ that your local changes are getting picked up. So, be sure to check the `Dockerfile` and if necessary, check for a `.docker/Dockerfile` that you might be able to copy from to make sure that your changes are used when generating the new image.

### \*\_USE\_VOLUME

By default, to go with the remote image that's used, a volume will be created to hold any changes that the container makes. This means that when local things change (such as uploading a file into a container), it goes into the volume, not locally on disk where the service is installed.

If you want to see these things locally, then set this to false.

### Changes to \*\_USE\_BUILD\_CONTEXT or \*\_USE\_VOLUME

If you make a change to either of these two variables, you need to rebuild the container to make them apply. Simply run `sudo ./mythic-cli build [name]` and you should see your changes.
