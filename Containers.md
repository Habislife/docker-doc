Throughout the course we have interacted with many different Docker objects. This module covers the additional options for how we can work with them.

Use `docker container --help` to see all the subcommands associated with Docker containers.

## Commands

### Attach

Attach local shell to a container's input, output, and error streams:

```bash
docker container attach container-id

# equivalent
docker attach container-id
```

### Exec

Run a new command within a container:

```bash
docker container exec container-id command

# equivalent
docker exec container-id command
```

### Inspect

Show detailed information about a container:

```bash
docker container inspect container-id
```

### Stop and Kill

Stop a container gracefully or forcefully:

```bash
docker container stop container-id

# equivalent
docker container stop container-id

docker container kill container-id

# equivalent
docker kill container-id
```

### Logs

View the logs of a container:

```bash
docker container logs container-id

# Add -f to tail the logs
docker container logs -f container-id

# equivalent
docker logs -f container-id
```

### List

List all running containers:

```bash
docker container ls

# Add -a to list stopped containers as well
docker container ls -a

# equivalent
docker ps -a
```

### Prune

Remove all stopped containers:

```bash
docker container prune
```

### Remove

Remove a specific container:

```bash
docker container rm container-id
```

### Run

Create a container from an image:

```bash
docker container run -it image-name:tag

# equivalent
docker run -it image-name:tag
```

### Top

See what's running inside a container:

```bash
docker container top container-id
```

### Wait

Wait for a container to finish before proceeding:

```bash
docker container wait container-id
```

This can be useful if you are scripting something that should only happen after a container exits.