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
`docker log <container>` | Print the output of the container.
`docker kill <container>` | Stop a running container.
`docker rm <container>` | Remove a container.
`docker run --memory <max-mem> <image> <command>` | Limit the memory used by the container.
`docker run --cpu-shares <cpu-rel> <image> <command>` | Limit the CPU relative % used by the container.
`docker run --cpu-quota <max-cpu> <image> <command>` | Limit the CPU absolute % used by the container.

## Exposing a network port

Command | Description
:-- | :--
`docker run -p in:out -p in2:out2` | Expose two TCP port from inside the container to outside.

# ID

Docker has two sets of `IDs`, one for images, and one for containers, that do not overlap.

_Container_ and _Images_ can be referenced in commands by NAMEs, TAGs, IDs, and also only the first characters of IDs.

---------

# Notes

- **1**: An example format for printing in multiple row, defined trough an environment variable, can be set running the command: ```export FORMAT="\nID\t{{.ID}}\nIMAGE\t{{.Image}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.RunningFor}}\nSTATUS\t{{.Status}}\nPORTS\t{{.Ports}}\nNAMES\t{{.Names}}\n"```
