# Docker Concepts
## Essential definitions

* Containers - An isolated environment run on the host operating system: includes a group of processes, a view of the OS image (see Docker images and filesystems below for more info), and a read-write layer that may be altered and is tied to the container.
* Image - In Docker, images are the filesystem that is used to construct the filesystem (including operating system files) used by the processes in a container. The image is often composed of multiple layers (see Fileystems below), but this is transparent to the user.
* Dockerfile - A recipe, much like a shell script, for creating a Docker Image. Can use other images as a “base” for create the new image.

## CPU sharing


There are several [different methods](https://docs.docker.com/engine/reference/run/) to limit CPU usage for containers. In the classroom setting, generally no alteration will be necessary if the Docker engine is being run on a single node, since much like regular processes, the CPU share mechanism will by default only enforce equal process access only when the node is fully utilizing its CPU(s). Even if multiple nodes are being employed, this should generally be sufficient.

In the case where you are being charged for individual containers, the advice changes, since you would want to limit the cpu quote for individual containers to minimize cost.

## Filesystems

The underlying system files that a docker container uses are stored in [read-only layers](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/). Any changes made to files in the read-only layers are stored in a thin read/write layer specific to the container in question. This results in minimal storage allocation (and thus start-up time as well) if a common operating system (Linux distribution) is chosen as the base for all container activities on a given system.


Containers that need to write a lot of data should consider using some form of [data volume](https://docs.docker.com/engine/tutorials/dockervolumes/) for increased performance, and where needed, for file persistence (e.g., project or homework output); while containers are persisted, an accidental ‘docker rm’, issued directly or as part of another script like ‘docker-compose down’ will be the end of the road for any files stored in the thin read/write layer for that container. Note that a data volume can be shared among multiple containers, so care should be taken to ensure that any possible clobbering of files in a shared data volume is acceptable or preventable.

# Docker Images in the Workshop
rstudio_workshop (RStudio)

This is primarily based on the [rocker/rstudio](https://hub.docker.com/r/rocker/rstudio/) image. We use a custom modification to the official rstudio image that allows for ssh-access and includes a few more utilities. This image is based on the Ubuntu Linux distribution.


## Using the image

### Build the image
Generally only need to do this once, or if you update your Dockerfile.

```
docker build -t rstudio_workshop .
```

### Start one or more containers.

We are running RStudio and SSH, these both require separate ports to be exposed on the host OS.

```
docker run -p <host OS RStudio port>:<container RStudio port> -p <host OS SSH port>:<container SSH port> -dit rstudio_workshop
```

This will start the container in of the specified image type (rstudio_workshop) in the background. A concrete example is:


```
docker run -p 8787:8787 -p 8622:22 -dit rstudio_workshop
```
This maps the container's port 8787 to the host OS port 8787 (they are the same). On the other hand, since the host OS is often
running SSH on port 22, we have to map it elsewhere: we map the container's port 22 to host OS port 8622.

Run `docker ps` to see the randomly generated name or container id of your image, which you will use in subsequent commands.

Now you can visit `http://<ip or hostname>:8787` in your browser to login to RStudio (username and password both rstudio), assuming
you used the port-mapping in the example above.


If you need to connect via ssh, first start the SSH server in the container:

```
docker exec -d <container name or ID> /usr/sbin/sshd
```

Then login either from the host machine (RStudio uses your Linux account credentials, so again, username and password are both 'rstudio'):

```
ssh rstudio@localhost -p 8622

```

or from a remote system, e.g., your desktop or laptop:

```
ssh rstudio@<host OS IP or hostname> -p 8622

```


# If you are on the host OS and want to connect without SSH, you can directly attach to a bash shell:

```
docker exec -it <container name or ID> /bin/bash
```

**Importantly, if you are running more than one container,** the host OS ports will need to be different for each container,
so that there are no collisions. For instance, to start another container on the same system, you could do:

```
docker run -p 8788:8787 -p 8623:22 -dit rstudio_workshop
```

where we bumped up the host OS ports by 1 in each case. As far as the container is aware, the ports remain the same, but to the outside
world, we are now using port 8788 for RStudio and port 8623 for SSH.


