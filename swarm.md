### How to promote `manager` by it IP from external.
```shell
docker-machine ssh manager "docker swarm init --advertise-addr $(docker-machine ip manager)"
```

### How to let `worker` join swarm mode by `manager` IP from external.
```shell
docker-machine ssh worker "docker swarm join --token `docker $(docker-machine config manager) swarm join-token worker -q` $(docker-machine ip manager)"
```
