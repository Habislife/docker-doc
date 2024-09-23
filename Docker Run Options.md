We saw the basic syntax for running containers earlier in the course:

```bash
docker run <CONTAINER_IMAGE_NAME>
```

This lesson provides a summary of the most important configuration options when running containers with Docker.

## Docker Compose vs Docker Run

There are two primary ways to run containers with Docker Desktop, `docker run` and `docker compose`.

For one-off containers, you can use `docker run` is sufficient.

That being said, docker compose allows you to specify all of your application configuration within a `YAML` file, making it more intuitive and easier to work with for applications with multiple containerized services.
![[Pasted image 20240923170357.png]]

## Common Docker Run Options

1. `-d` (Detach): Run a container in the background.

```bash
docker run -d ubuntu sleep 5
```

2. `--entrypoint` (Entry Point): Override the entry point defined in the Dockerfile.

```bash
docker run --entrypoint echo ubuntu hello
```

3. `--env` or `-e` (Environment Variables): Set environment variables at runtime

```bash
docker run --env MY_ENV=hello ubuntu printenv
```

4. `--init` (Initialization): Run Docker's initialization script and spawn the process as a subprocess.

```bash
docker run --init ubuntu ps
```

5. `-i` (Interactive) and `-t` (TTY): Have an interactive TTY session inside the container.

```bash
docker run -it ubuntu
```

6. `--mount` and `--volume` (Volume): Persist data outside of the container layer in a volume.

```bash
docker run \
  -e POSTGRES_PASSWORD=foobarbaz \
  --volume pgdata:/var/lib/postgresql/data \
  postgres:15.1-alpine
```

7. `--name` (Name): Provide a specific name for a container.

```bash
docker run -d --name my_container ubuntu sleep 99
```

8. `--network` or `--net` (Network): Connect to a specific Docker network.

```bash
docker run --network my_network ubuntu
```

9. `--platform` (Platform): Specify the architecture to run the container image.

```bash
docker run --platform linux/arm64/v8 ubuntu dpkg --print-architecture
```

10. `--publish` or `-p` (Publish): Connect a port from the host system to that of the container.

```bash
docker run -p 3000:3000 api-node
```

11. `--restart` (Restart): Restart the container based on the specified policy (always, unless-stopped, or never).

````bash
docker run --restart unless-stopped ubuntu
```

12. `--rm` (Remove): Remove the container when the process exits.
```bash
docker run --name this_one_will_remain ubuntu
docker run --rm --name this_one_will_be_gone ubuntu

# grepping for these containers shows that the --rm one is gone
docker image ls -a | grep this_one_will
````

## Advanced Configuration Options

1. `--cap-add` and `--cap-drop`: Specify which Linux capabilities should be accessible from the container.
2. `--cgroup-parent`: Specify which cgroup ID the container should be associated with.
3. `--cpu-shares`: Specify the percentage of CPU cycles the container should have access to.
4. `--cpuset`: Specify which CPU cores the container should run on.
5. `--device-read-bps` and `--device-write-bps`: Control the device throughput and bandwidth the container has access to.
6. `--gpus`: Access GPUs within the container.
7. `--health-*` (e.g. `--health-cmd`, `--health-interval`, etc...): Specify a health check for Docker to periodically ping the container.
8. `--memory`: Specify the amount of memory the container process should have access to.
9. `--pids-limit`: Specify the number of subprocesses the container should be allowed to manage.
10. `--privileged`: Grant the container access to all privileges.
11. `--read-only`: Set the container layer of the file