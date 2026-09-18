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

_docker run -d -p hostport:containerport namespace/name:tag_

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

- _docker stop_: This stops the container by issuing a SIGTERM signal to the container. You'll typically want to use docker stop.
- _docker kill_: This stops the container by issuing a SIGKILL signal to the container. This is a more forceful way to stop a container, and should be used as a last resort.

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

The docker run command has a _--network none_ flag that makes it so that the container can't network with the outside world, which is super useful for isolating containers.

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
curl http://localhost:8880/
```

## Dockerfiles

### Dockerfile

```Dockerfile
# This is a comment

# Use a lightweight debian os
# as the base image
FROM debian:stable-slim

# execute the 'echo "hello world"'
# command when the container runs
CMD ["echo", "hello world"]
```

```sh
docker build . -t helloworld:latest
docker run helloworld
docker ps -a
```

### Building a server

```sh
go mod init github.com/mariolazzari/bd-docker
go build
```

### Dockerizing server

```Dockerfile
FROM debian:stable-slim

# COPY source destination
COPY bd-docker /bin/goserver

CMD ["/bin/goserver"]
```

```sh
docker build --platform linux/amd64 -t goserver:latest .
docker run --rm -p 8010:8010 goserver:latest
```

### Enviroment vars

```Dockerfile
FROM debian:stable-slim

ENV PORT=8991

# COPY source destination
COPY bd-docker /bin/goserver

CMD ["/bin/goserver"]
```

```sh
GOOS=linux GOARCH=amd64 go build
docker build . -t goserver:latest
docker run -p 8991:8991 goserver
```

### Dockerizing Python

```Dockerfile
# Build from a slim Debian/Linux image
FROM debian:stable-slim

# Update apt
RUN apt update
RUN apt upgrade -y

# Install build tooling
RUN apt install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev

# Download Python interpreter code and unpack it
RUN wget https://www.python.org/ftp/python/3.10.8/Python-3.10.8.tgz
RUN tar -xf Python-3.10.*.tgz

# Build the Python interpreter
RUN cd Python-3.10.8 && ./configure --enable-optimizations && make && make altinstall

# Copy our code into the image
COPY main.py main.py

# Copy our data dependencies
COPY books/ books/

# Run our Python script
CMD ["python3.10", "main.py"]
```

```sh
docker build -t bookbot -f Dockerfile.py .
docker run bookbot
```

## Debug

### Docker logs

```sh
docker logs [OPTIONS] CONTAINER
docker run -d --name logdate alpine sh -c 'while true; do echo "LOGGING: $(date)"; sleep 1; done'
docker ps
docer logs
# realtime logs
docker logs -f logdate
docker logs --tail 5 CONTAINER
```

### Stats

```sh
docker stats [OPTIONS] CONTAINER
docker run -d --name cpu-stress alexeiled/stress-ng --cpu 2 --timeout 10m
docker run -d --name mem-stress alexeiled/stress-ng --vm 1 --vm-bytes 1G --timeout 10m
docker stats
```

### Top

```sh
docker top CONTAINER
docker top CONTAINER [ps OPTIONS]
# Check the processes in the CPU-intensive container
docker top cpu-stress
# Check the processes in the memory-intensive container
docker top mem-stress
```

### Resource Limits

```sh
docker run -d --cpus="0.25" --name cpu-stress alexeiled/stress-ng --cpu 2 --timeout 10m
docker stats
```

## Publish

### Publishing images

Docker Hub is the official cloud service for storing and sharing Docker images.
We call these kinds of services "registries".
Other popular image registries include:

- AWS ECR
- GCP Artifact Registry
- GitHub Container Registry
- Harbor
- Azure ACR

```sh
GOOS=linux GOARCH=amd64 go build
docker build . -t mariolazzari/goserver
docker run -p 8991:8991 mariolazzari/goserver
docker push mariolazzari/goserver
```

### Delete and Pull

```sh
docker image rm mariolazzari/goserver
docker pull mariolazzari/goserver
docker run -p 8991:8991 mariolazzari/goserver
```

### Tags

```sh
docker build . -t mariolazzari/goserver:0.2.0
docker run -p 8991:8991 mariolazzari/goserver:0.2.0
docker push mariolazzari/goserver:0.2.0
docker pull mariolazzari/goserver:0.2.0
docker run -p 8991:8991 mariolazzari/goserver:0.2.0
```

### Latest

```sh
docker build -t bootdotdev/awesomeimage:5.4.6 -t bootdotdev/awesomeimage:latest .
docker push bootdotdev/awesomeimage --all-tags
```
