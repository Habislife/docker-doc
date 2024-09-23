It is useful to break down the various components within the Docker ecosystem.

Docker Desktop is an application you install on development systems that provides:

- **A client application:**
    - _**Command Line Interface (CLI):**_ Interact with Docker using commands like`docker run` or `docker pull`.
    - _**Graphical User Interface (GUI):**_ Browse images, configure CPU, memory, and disk space allocation.
    - _**Credential Helper:**_ Store credentials for private registries.
    - _**Extensions:**_ Third-party software that provides additional functionality.
- **A Linux virtual machine containing:**
    - **Docker daemon (dockerd):** Manages container objects, networking, and volumes within the server host application. It exposes the Docker API for communication with the client application.
    - **(Optional) Kubernetes cluster**
    ![[Pasted image 20240923155239.png]]

## Docker Engine vs Docker Desktop

There is often confusion between "Docker Desktop" and "Docker Engine". Docker Engine refers specifically to a subset of the Docker Desktop components which are free and open source and can be installed only on Linux.

Docker Engine includes:

- Docker Command Line Interface (CLI)
- Docker daemon (dockerd), exposing the Docker Application Programming Interface (API)

Docker Engine can build container images, run containers from them, and generally do most things that Docker Desktop but is Linux only and doesn't provide all of the developer experience polish that Docker Desktop provides.

For more information about docker engine see: [https://docs.docker.com/engine/](https://docs.docker.com/engine/)

## Container Registries

Container image registries are not part of Docker itself, but because they are the primary mechanism for storing and sharing container images it is worth including it here. Docker runs a registry named DockerHub, but there are many other registries as well. More info on these can be found in [[Container Registries]].