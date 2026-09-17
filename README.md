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
