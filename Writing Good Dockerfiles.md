## General Process

Dockerfiles generally have steps that are similar to those you would use to get your application running on a server.

```
1. Start with an Operating System
2. Install the language runtime
3. Install any application dependencies
4. Set up the execution environment
5. Run the application
```

**Note:** We can often jump right to #3 by choosing a base image that has the OS and language runtime preinstalled.

## Useful Techniques:

Here are some of the techniques demonstrated in the Dockerfiles within this repo:

1. **Pinning a specific base image:** By specifying an image tag, you can avoid nasty surprises where the base image
2. **Choosing a smaller base image:** There are often a variety of base images we can choose from. Choosing a smaller base image will usually reduce the size of your final image.
3. **Choosing a more secure base image:** Like image size, we should consider the number of vulnerabilities in our base images and the attack surface area. Chaingaurd publishes a number of hardened images (https://www.chainguard.dev/chainguard-images).
4. **Specifying a working directory:** Many languages have a convention for how/where applications should be installed. Adhering to that convention will make it easier for developers to work with the container.
5. **Consider layer cache to improve build times:** By undersanding the layered nature of container filesytems and choosing when to copy particular files we can make better use of the Docker caching system.
6. **Use COPY —link where appropriate:** The `--link` option was added to the `COPY` command in march 2022. It allows you to improve cache behavior in certain situations by copying files into an independent image layer not dependent on its predecessors.
7. **Use a non-root user within the container:** While containers can utilize a user namespace to differentiate between root inside the container and root on the host, this feature won't always be leveraged and by using a non-root user we improve the default safety of the container. When using Docker Desktop, the Virtual Machine it runs provides an isolation boundary between containers and the host, but if running Docker Engine it is useful to use a user namespace to ensure container isolation ([more info here](https://docs.docker.com/engine/security/userns-remap/)). This page also provides a good description for why to avoid running as root ([container best practices](https://cloud.google.com/architecture/best-practices-for-operating-containers#avoid_running_as_root)).
8. **Specify the environment correctly:** Only install production dependencies for a production image, and specify any necessary environment variables to configure the language runtime accordingly.

## Impact of Applying These Techniques

In general, these techniques impact some combination of (1) build speed, (2) image security, and (3) developer clarity. The following summarizes these impacts:

```
Legend:
 🔒 Security
 🏎️ Build Speed
 👁️ Clarity
```

- Pin specific versions [🔒 👁️]
    - Base images (either major+minor OR SHA256 hash) [🔒 👁️]
    - System Dependencies [🔒 👁️]
    - Application Dependencies [🔒 👁️]
- Use small + secure base images [🔒 🏎️]
- Protect the layer cache [🏎️ 👁️]
    - Order commands by frequency of change [🏎️]
    - COPY dependency requirements file → install deps → copy remaining source code [🏎️]
    - Use cache mounts [🏎️]
    - Use COPY --link [🏎️]
    - Combine steps that are always linked (use heredocs to improve tidiness) [🏎️ 👁️]
- Be explicit [🔒 👁️]
    - Set working directory with WORKDIR [👁️]
    - Indicate standard port with EXPOSE [👁️]
    - Set default environment variables with ENV [🔒 👁️]
- Avoid unnecessary files [🔒 🏎️ 👁️]
    - Use .dockerignore [🔒 🏎️ 👁️]
    - COPY specific files [🔒 🏎️ 👁️]
- Use non-root USER [🔒]
- Install only production dependencies [🔒 🏎️ 👁️]
- Avoid leaking sensitive information [🔒]
- Leverage multi-stage builds [🔒 🏎️]