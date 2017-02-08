### To understand volumes (will convert to md later)
- http://www.tricksofthetrades.net/2016/03/14/docker-data-volumes/

### To remove existing container by image.
```shell
docker rm -f $(docker ps -a -q --filter ancestor=rabbotio/nap-app)
```

---

[Source](https://codefresh.io/blog/everyday-hacks-docker/ "Permalink to Everyday Hacks for Docker - Codefresh")

### Cleaning Up
> After working with Docker for some time, you start accumulating development junk: unused volumes, networks, exited containers and unused images.

```shell
docker system prune
```

### Remove Dangling Volumes
> `dangling` volumes are volumes not in use by any container.

```shell
docker volume rm $(docker volume ls -q -f "dangling=true")
```

### Remove Exited Containers
> First, list the containers (only IDs) you want to remove (with filter) and then remove them (consider `rm -f` to force remove).

```shell
docker rm $(docker ps -q -f "status=exited")
```

### Remove Dangling Images
> `dangling` images are untagged images, that are the leaves of the images tree (not intermediary layers).

```shell
docker rmi $(docker images -q -f "dangling=true")
```

### Autoremove Interactive Containers
> When you run a new interactive container and want to avoid typing `rm` command after it exits, use `\--rm` option. Then when you exit from the created container, it will be automatically destroyed.

```shell
docker run -it --rm alpine sh
```

### Inspect Docker Resources
```shell
# Pretty JSON and jq Processing
brew install jq

# show whole Docker info
docker info --format "{{json .}}" | jq .
 
# show Plugins only
docker info --format "{{json .Plugins}}" | jq .
 
# list IP addresses for all containers connected to 'bridge' network
docker network inspect bridge -f '{{json .Containers}}' | jq '.[] | {cont: .Name, ip: .IPv4Address}'
```

### Watching Containers Lifecycle
> Sometimes you want to see containers being activated and exited when you run certain docker commands or try different restart policies. The[`watch`][5] command combined with [`docker ps`][6] can be pretty useful here. I found that the `docker stats` command (even with `\--format` option) is not useful for this because it doesn't allow you to see the same information as you can with the `docker ps` command.

```shell
# Display a Table with ‘ID Image Status’ for Active Containers and Refresh it Every 2 Seconds
watch -n 2 'docker ps --format "table {{.ID}}\t {{.Image}}\t {{.Status}}"'
```

### Enter into Host/Container Namespace
```shell
# get a shell into Docker host
docker run --rm -it --privileged --pid=host walkerlee/nsenter -t 1 -m -u -i -n sh
```

### Enter into ANY Container
```shell
# get a shell into 'redis' container namespace
docker run --rm -it --privileged --pid=container:redis walkerlee/nsenter -t 1 -m -u -i -n sh
```

### Create Alpine Based Container with ‘htop’ Tool
```shell
docker build -t htop - << EOF
FROM alpine
RUN apk --no-cache add htop
EOF
```

### Docker Command Completion
```shell
# Tap homebrew/completion to gain access to these
brew tap homebrew/completions
 
# Install completions for docker suite
brew install docker-completion
brew install docker-compose-completion
brew install docker-machine-completion
```

### Start Containers Automatically
```shell
# Restart Always
docker run --restart=always redis

# Restart Container on Failure
docker run --restart=on-failure:10 redis
```

### Network Tricks
> There are times when you might want to create a new container and connect it to an existing network stack. This might be the Docker host network or another container’s network. This is helpful when debugging and auditing network issues.
The docker run `--network/net option` allows you to do this.

```shell
# Use the Docker Host Network Stack
docker run --net=host ...

# Use Another Container’s Network Stack
docker run --net=container:<name|id> ...
```

### Attachable Overlay Network
> Using Docker Engine running in swarm mode, you can create a multi-host `overlay` network on a manager node. When you create a new swarm service, you can attach it to the previously created `overlay` network.

> Sometimes you need to attach a new Docker container (filled with different networking tools), to an existing `overlay` network, in order to inspect the network configuration or debug network issues.  You can use the `docker run` command for this, eliminating the need to create a completely new “debug service”.

> Docker 1.13 brings a new option to the `docker network create` command: `attachable`. The `attachable` option enables manual container attachment.

```shell
# create an attachable overlay network
docker network create --driver overlay --attachable mynet

# create net-tools container and attach it to mynet overlay network
docker run -it --rm --net=mynet net-tools sh
```
