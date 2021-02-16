# Docker Cheatsheet

## Handling images

Command | Description
:-- | :--
`docker images` | List all the images present in the system.
`docker run -ti <image> <command>` | Create a container, by run a command in an image, with a _'terminal interactive'_ so command are captured from the keyboard and print on the screen.
`docker ps --format $FORMAT` | Have a look at the running images, also stopped, using a custom format $^1$.
`docker ps -a` | Show stopped containers.
`docker ps -l` | Show the last container that exit (a stopped container).
`docker commit <container>` | Create an image, from a container (as opposed to run), and output its sha256 ID.
`docker tag <image-id> <tag-name>` | Associate a tag name to a newly created image.

## Run process

Command | Description
:-- | :--
`docker run -ti --rm <image> <command>` | Create a container, run the command, use stdio (`-ti`), remove after (`--rm`).
`docker run -ti --rm <image> bash -c "command 1; command 2 && command 3"` | Run multiple commands.
`docker run -d -ti <image> <command>` | Run the container and detach from it.
`docker attach <container>` | Connect to a running container.
`ctrl-p ctrl-q` | Detach a running container.
`docker exec -ti <image> <command>` | Run another command in a running container.
`docker run --name <name> -d <image> <command>` | Create a container with a specified name.

## Manage containers

Command | Description
:-- | :--
`docker ps` | List the running containers. 
`docker log <container>` | Print the output of the container.
`docker stop <container>` | Stop a running container using SIGTERM.
`docker kill <container>` | Stop a running container using SIGKILL.
`docker rm <container>` | Remove a container.
`docker run --memory <max-mem> <image> <command>` | Limit the memory used by the container.
`docker run --cpu-shares <cpu-rel> <image> <command>` | Limit the CPU relative % used by the container.
`docker run --cpu-quota <max-cpu> <image> <command>` | Limit the CPU absolute % used by the container.

## Exposing a network port

Command | Description
:-- | :--
`docker run -p <in-port>:<out-port> <image> <command>` | Expose a TCP port from inside the container to outside.
`nc host.docker.internal <port>` | Access server port from inside the container (on MAC OS).
`docker run -p <in-port> <image> <command>` | Expose a TCP port from inside the container, but let docker choose the outside port.
`docker port <container>` | Print the assigned port of the container.
`docker run -p <in-port>:<out-port>/udp <image> <command>` | Expose an UDP port from inside the container to outside.

## Working with Networks

By default docker has three network available:

- _bridge_: default network used by containers
- _host_: expose direct access of the host network to the container
- _none_: do not allow network connections at all

Container names are important, since can also be used in network commands to access other containers, like `ping <container_name>`.

Command | Description
:-- | :--
`docker network ls` | List available networks.
`docker network create <net_name>` | Create a network.
`docker run --rm -ti --net <net_name> --name <name>` | Give container access to the network.
`docker network connect <net_name> <container>` | Connect a container to a network.

## Legacy Linking

- _Legacy Linking_ will link all ports one way.
- Also _secret environment variables_ are shared only one way.
- This is not anymore used, and create dependencies on the startup order.
- Restart sometimes break the link.

Command | Description
:-- | :--
`docker run --rm -ti -e TST=blabla --name serverA <image> <command>` | Create a container with an environment variale TST.
`docker run --rm -ti --link serverA --name serverB <image> <command>` | Create a container and link it to the one named _serverA_. ServerA _cannot_ connect to the serverB, but ServerB can connect to ServerA and also read the ENV as SERVERA_ENV_TST.

## Images

The tag of an image taken from registry server host:port, should have this structure:
_<registry_host>:<port>/<organization>/<image-name>:version-tag_.

Command | Description
:-- | :--
`docker images` | List all the images downloaded.
`docker commit <container> <image>:<tag>` | Create a new image from the container. If not specified the TAG will be equal to latest.
`docker pull <image>:<ver>` | Pull an image from a registry.
`docker tag <image>:<ver> <repo>/<image>:<ver>` | Retag a local image
`docker push <repo>/<image>:<ver>` | Push a new image to a registry.
`docker rmi <image>:<ver>` | Remove an image.

## Volumes

There are two types of volumes:

- _Persistent_: data is available also after the container goes away.
- _Ephemeral_: when no container is using them, the disappear.

Command | Description
:-- | :--
`docker run -ti -v <local_path>:<container_path> <image> <command>` | Create a _persistent_ volume from a folder.
`docker run -ti -v <local_file>:<container_file> <image> <command>` | Create a _persistent_ volume from a file, that should exist _before_ the container is created.
`docker run -ti -v <volume_name> <image> <command>` | Create a container with an _ephemeral_ volume.
`docker run -ti --volumes-from <container>  <image> <command>` | Create a container that will use the _ephemeral_ volume of another container.

## Registries

Command | Description
:-- | :--
`docker search ubuntu` | Search images, same as hub.docker.com.
`docker login` | Login into the docker hub registry.
`docker push <image>` | Send an image to the docker hub registry.

## Dockerfile

Image creation can be automated using a [dockerfile](https://docs.docker.com/engine/reference/builder/). Each line is a call to `docker run` and `docker commit`, so a process started on one line will not be running on the next line.

Argument to execute can be gine in two forms:
- _shell form_: is the command that will be passed to a shell (e.g. nano notes.txt)
- _exec form_: instruct to directly run the command, without a shell (eg. ["/bin/nano", "notes.txt"])

Command | Description
:-- | :--
`docker build -t name-of-result .` | Build an image following the instruction in the Dockerfile.
`FROM` | Select from witch image to download to start from
`MANTAINER Firstname Lastname <email_address>` | The mantainer name of the image (the email should contain the angled brackets).
`RUN command` | Run the command line into the container.
`ADD <source> <destination>` | Add the file to the image. If is an archive .tar.gz, automatically uncompressed it in the given path. And can also be an URL.
`ENV var=value` | Set an environment variable both during the build and when running the result.
`ENTRYPOINT` | Specifies the start of the command to run.
`CMD <command>` | Specify the command to run.
`EXPOSE 8080` | Expose a port of a container.
`VOLUME ["/shared-data"]` | Create a volume that can be inherited by later containers. Should be avoided.
`VOLUME ["/host/path/" "/container/path"]` | Create a persistent volume.
`WORKDIR` | Set the directory from where all the rest of RUN command will be executed.
`USER <user>` | Execute command as user with the given name or id.

The simplest possible example is:
```
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get -y install nano
ADD notes.txt /tmp/notes.txt
CMD ["/bin/nano", "/tmp/notes.txt"]
```

A more complex example can be a _Multi-project Docker file_ that will use multiple FROM commands.

## Control Socket

Docker client and server communicate trough the socket `/var/run/docker.sock`. The docker server make use and let be accessible some special features of the linux kernel:

- _cgroups_
- _namespaces_
- _copy-on-write_

It is possible to run a docker client inside a docker image, further automating the process of creating containers.

Command | Description
:-- | :--
`docker run -ti --rm -v /var/run/docker.sock:/var/run/docker.sock docker sh` | Run the docker client inside a container and give access to the host control socket.
`docker info` | For check, inside the container, if docker is accessible and is running.

## Networks and Bridges

We can take a look at the bridge created by docker for handling private networks, starting a container with the `brctl` program installed and complete access to the network disabling network isolation (unsafe):

```
docker run -ti --rm --net=host ubuntu:16.04 bash
apt-get update && apt-get install -y bridge-utils
brctl show
```

We can also have a look of the firewall rules inserted by docker when enabling port forward, starting a container with privileged mode and complete host access:

```
docker run -ti --rm --net=host --privileged=true ubuntu bash
apt-get update && apt-get install -y iptables
sudo iptables -n -L -t nat
```

## Processes

Command | Description
:-- | :--
`docker inspect --format '{{.State.Pid}}' hello` | Get the PID of the container with name hello.
`docker run -ti --rm --privileged=true --pid=host ubuntu bash` | Start a privileged container with direct access to host processes.
## ID

Docker has two sets of `IDs`, one for images, and one for containers, that do not overlap.

_Container_ and _Images_ can be referenced in commands by NAMEs, TAGs, IDs, and also only the first characters of IDs.

## Registries

Before exposing a local registry to the internet, we should enforce security as described on https://docs.docker.com/registry.

There are also other options:

- The [Docker Trusted Registry](https://docs.docker.com/ee/dtr)
- The [AWS Elastic Container Registry](https://aws.amazon.com/ecr/)
- The [Google Cloud Container Registry](https://cloud.google.com/container-registry)
- The [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry)

Command | Description
:-- | :--
`docker run -d -p 5000:5000 --restart=always --name registry registry:2` | Run a registry server inside a docker container, and let the container restart if the program crash, using:
`docker tag ubuntu:14.04 localhost:5000/mycompany/my-ubuntu:99` | I tag an image
`docker push localhost:5000/mycompany/my-ubuntu:99` | I save in my repository the new image
`docker save -o my-image.tar.gz debian:sid busybox ubuntu:14.04` | Save images into an archive.
`docker load -i my-images.tar.gz` | Load images from an archive.

## Orchestrator

They are systems for start containers, keep the running, and restart them if they fail. There are many option available, like:

- _Docker Compose_: useful for developing and testing, but not for production environment.
- _Kubernetes_: assemble containers into pods, allow scaling and management.
- _AWS EC2 Container Service_: analogous to Kubernetes, with different vocabulary.

One example about Kubernetes can be found at https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html.

---------

# Notes

- **1**: An example format for printing in multiple row, defined trough an environment variable, can be set running the command: ```export FORMAT="\nID\t{{.ID}}\nIMAGE\t{{.Image}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.RunningFor}}\nSTATUS\t{{.Status}}\nPORTS\t{{.Ports}}\nNAMES\t{{.Names}}\n"```
