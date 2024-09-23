Throughout the course we have interacted with many different Docker objects. This module covers the additional options for how we can work with them.

Use `docker volume --help` to see all the subcommands associated with Docker volumes.

## Commands

### Create

Create a new volume:

```bash
docker volume create my-volume
```

### Inspect

Inspect a volume:

```bash
docker volume inspect my-volume
```

### List

List all available volumes:

```bash
docker volume ls
```

### Prune

Remove all unused volumes:

```bash
docker volume prune
```

### Remove

Remove one or more volumes:

```bash
docker volume rm my-volume
```