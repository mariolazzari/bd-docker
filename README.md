# Learn Docker

## Install Docker

### Docker Desktop

[MacOS](https://docs.docker.com/desktop/setup/install/mac-install/)
[Linux](https://docs.docker.com/desktop/setup/install/linux/)
[Windows](https://docs.docker.com/desktop/setup/install/windows-install/)

```sh
docker version
```

### Docker Hub

```sh
docker login
```

## Containers

### Images

```sh
docker pull docker/getting-started
docker images
```

### Run a container

*docker run -d -p hostport:containerport namespace/name:tag*

- -d: Run in detached mode (doesn't block your terminal)
- -p: Publish a container's port to the host (forwarding)
- hostport: The port on your local machine
- containerport: The port inside the container
- namespace/name: The name of the image (usually in the format username/repo)
- tag: The version of the image (often latest)

```sh
docker run -d -p 8965:80 docker/getting-started:latest
docker ps
```

### Stop container

- *docker stop*: This stops the container by issuing a SIGTERM signal to the container. You'll typically want to use docker stop.
- *docker kill*: This stops the container by issuing a SIGKILL signal to the container. This is a more forceful way to stop a container, and should be used as a last resort.

```sh
docker ps
docker op 8cd796458931
```

## Storage

### Volumes

```sh
docker volume create ghost-vol
docker volume ls
docker volume inspect ghost-vol
```

### Run Ghost

```sh
docker pull ghost
docker run -d \
  -e NODE_ENV=development \
  -e url=http://localhost:3001 \
  -e database__connection__filename=/var/lib/ghost/content/data/ghost-dev.db \
  -p 3001:2368 \
  -v ghost-vol:/var/lib/ghost/content \
  ghost
```

### Persist

- A container's file system is read-write, but when you delete a container, and start a new one from the same image, that new container starts from scratch again with a copy of the image. All stateful changes are lost.
- A volume's file system is read-write, but it lives outside a single container. If a container uses a volume, then stateful changes can be persisted to the volume even if the container is deleted.


```sh
docker ps
docker stop CONTAINER_ID
docker rm CONTAINER_ID
docker run -d \
  -e NODE_ENV=development \
  -e url=http://localhost:3001 \
  -e database__connection__filename=/var/lib/ghost/content/data/ghost-dev.db \
  -p 3001:2368 \
  -v ghost-vol:/var/lib/ghost/content \
  ghost
```

### Delete volume

```sh
docker ps -a
docker run -d \
  -e NODE_ENV=development \
  -e url=http://localhost:3001 \
  -e database__connection__filename=/var/lib/ghost/content/data/ghost-dev.db \
  -p 3001:2368 \
  -v ghost-vol:/var/lib/ghost/content \
  ghost
docker volume ls
docker stop <id>
docker rm <id>
docker volume rm ghost-vol
```

### Cleanup

```sh
docker ps -a
docker volume ls
```

## Help

### Execute

```sh
docker --help
docker help
```

### Exec

```sh
docker ps
docker run -d -p 8965:80 docker/getting-started
docker ps
docker exec CONTAINER_ID ls
docker exec f5b90270df5f  touch hacker.log
docker exec f5b90270df5f  ls
```

### Exec Netstat

```sh
docker exec f5b90270df5f netstat -ltnp
```

### Live shell

```sh
docker exec -it CONTAINER_ID /bin/sh
exit
```

## Networks

### Offline

The docker run command has a *--network none* flag that makes it so that the container can't network with the outside world, which is super useful for isolating containers.

### Break network

```sh
docker ps -a
docker run -d --network none docker/getting-started
docker exec CONTAINER_ID ping google.com -W 2
```

### Load balancers

A load balancer behaves as advertised: it balances a load of network traffic across some number of servers.

### Application Servers

```sh
docker pull caddy
docker run -d -p 8881:80 -v $PWD/index1.html:/usr/share/caddy/index.html caddy
docker run -d -p 8882:80 -v $PWD/index2.html:/usr/share/caddy/index.html caddy
```

### Custom Network

```sh
docker network create caddytest
docker network ls
docker run -d -p 8881:80 -v $PWD/index1.html:/usr/share/caddy/index.html caddy

docker network create caddytest
docker network ls

docker run --name -d -p 8965:80 docker/getting-started
docker run -d --name caddy1 --network caddytest -v $PWD/index1.html:/usr/share/caddy/index.html caddy
docker run -d --name caddy2 --network caddytest -v $PWD/index2.html:/usr/share/caddy/index.html caddy
docker run -it --network caddytest docker/getting-started /bin/sh

#docker run --name caddy1 -d -p 8881:80 -v $PWD/index1.html:/usr/share/caddy/index.html caddy
#docker run --name caddy2 -d -p 8882:80 -v $PWD/index2.html:/usr/share/caddy/index.html caddy
```

### Load ballancer config

```sh
localhost:80

reverse_proxy caddy1:80 caddy2:80 {
    lb_policy       round_robin
}
```

```sh
docker run -d --network caddytest -p 8880:80 -v $PWD/Caddyfile:/etc/caddy/Caddyfile caddy
```
