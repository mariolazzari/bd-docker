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
