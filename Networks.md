Throughout the course we have interacted with many different Docker objects. This module covers the additional options for how we can work with them.

Use `docker network --help` to see all the subcommands associated with Docker networks.

## Commands

### Create

Create a new network:

```bash
docker network create my-network
```

### Inspect

Inspect a network:

```bash
docker network inspect my-network
```

### List

List all networks:

```bash
docker network ls
```

### Prune

Remove all unused networks:

```bash
docker network prune
```

### Remove

Remove a specific network:

```bash
docker network rm my-network
```

### Connect

Connect a container to a network:

```bash
docker network connect my-network container-id
```

### Disconnect

Disconnect a container from a network:

```bash
docker network disconnect my-network container-id
```