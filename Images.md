Throughout the course we have interacted with many different Docker objects. This module covers the additional options for how we can work with them.

Use `docker image --help` to see all the subcommands associated with Docker image.

## Commands

### Build

Build an image from a Dockerfile:

```bash
docker image build -t new-image .

# equivalent
docker build -t new-image .
```

### History

Show the steps used to build an image:

```bash
docker image history new-image
```

### Inspect

Show detailed information about an image:

```bash
docker image inspect new-image
```

### Import
Create an image from a tarball:

```bash
docker image import file.tar
```

### Load

Create an image from a tar archive generated using docker save:

```bash
docker image load -i file.tar
```

### List

List all images on the system:

```bash
docker image ls
```

### Prune

Clean up old images:

```bash
docker image prune
```

### Pull

Pull an image from a registry:

```bash
docker image pull image-name:tag
```

### Push

Push built images to a registry:

```bash
docker image push image-name:tag
```

### Remove

Remove a specific image:

```bash
docker image rm new-image
```

### Save

Save an image to a tar archive:

```bash
docker image save -o file.tar image-name:tag
```

### Tag

Tag an image with a new tag:

```bash
docker image tag ubuntu:22.04 my-ubuntu-image
```

### Scan

Scan an image for known vulnerabilities:

```bash
docker scan ubuntu:22.04
```

**NOTE:** `docker scan` has been deprecated in favor of `docker scout` [https://docs.docker.com/scout/](https://docs.docker.com/scout/)

```
docker scout cves ubuntu:22.04
```