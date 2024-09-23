A container registry is a repository or a collection of repositories used to store and access container images. These registries are generally stored in the cloud and allow you to push built images and pull them into your deployment environment. Container registries can be either public or private. Some popular container registries include Docker Hub, GitHub Container Registry, and those provided by Google, Amazon, and Azure.

TODO: INSERT_IMAGE

## Authentication and Pushing Images

To push an image to a remote repo, you need to:

1. Authenticate to the repo
2. Tag the image with a tag corresponding to the repo

Each registry will have its own set of instructions for logging in. The process will generally involve using a command like `docker login` and providing your credentials.

## Example: Pushing to Docker Hub

1. Authenticate with Docker Hub:

```bash
docker login
```

2. Tag your image with your Docker Hub username and the repository name:

```bash
docker tag my_scratch_image:latest myusername/my_scratch_image:latest
```

3. Push the image to DockerHub:

```bash
docker push myusername/my_scratch_image:latest
```

## Example: Pushing to GitHub Container Registry

1. Authenticate with GitHub Container Registry:

```bash
echo "TOKEN" | docker login ghcr.io -u USERNAME --password-stdin
```

2. Tag your image with the registry prefix, your GitHub username, and the repository name:

```bash
docker tag my_scratch_image:latest ghcr.io/myusername/my_scratch_image:latest
```

3. Push the image to GitHub Container Registry:

```bash
docker push ghcr.io/myusername/my_scratch_image:latest
```

### Tagging Best Practices

- Avoid using the latest tag, and instead use descriptive tags for your images.
- The same image can have multiple tags, so use tags that are useful for the end user.
- Treat tags as immutable, except for temporary tags used for development.
- Common tag elements include timestamps, build IDs, commit hashes, and semver release versions.