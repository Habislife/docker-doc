There are some additional features of Dockerfiles that are not shown in the example applications but are worth knowing about. These are highlighted in `Dockerfile.sample` and the corresponding build / run commands in the `Makefile`

```bash
# syntax=docker/dockerfile:1.5
# escape=\
# ^ OPTIONAL "directives" (must be at top if used)

# THIS IS A COMMENT

# ARG is the only instruction that can come before FROM
ARG BASE_IMAGE_TAG=19.4
# ARGs can be overriden at build time
# > docker build --build-arg BASE_VERSION=19.3 .

FROM node:${BASE_IMAGE_TAG}

LABEL org.opencontainers.image.authors="sid@devopsdirective.com"

RUN echo "Hey Team 👋 (shell form)"
RUN ["echo", "Hey Team 👋 (exec form)"]

# Heredocs allow for specifying multiple commands to
# be run within a single step, across multiple lines
# without lots of && and \
RUN <<EOF
apt update
apt install iputils-ping -y
EOF

# --mount allows for mounting additional files
# into the build context
# RUN --mount=type=bind ...
# RUN --mount=type=cache ...
# RUN --mount=type=ssh ...
RUN --mount=type=secret,id=secret.txt,dst=/container-secret.txt \
  echo "Run the command that requires access to the secret here"

# Available only at build time
# (Still in image metadata though...)
ARG BUILD_ARG=foo

# Available at build and run time
ENV ENV_VAR=bar

# Set the default working directory
# Use the convention of your language/framework
WORKDIR path/to/the/working/directory

ENTRYPOINT [ "echo", "Hey Team 👋 (entrypoint)" ]
CMD [ "+ (cmd)" ]
```

1. **Parser directives:** Specify the particular Dockerfile syntax being used or modify the escape character.
2. **ARG:** Enables setting variables at build time that do not persist in the final image (but can be seen in the image metadata).
3. **Heredocs syntax:** Enables multi-line commands within a Dockerfile.
4. **Mounting secrets:** Allows for providing sensitive credentials required at build time while keeping them out of the final image.
5. **ENTRYPOINT + CMD:** The interaction between `ENTRYPOINT` and `CMD` can be confusing. Depending on whether arguments are provided at runtime one or more will be used.
    - See the examples by running `make run-sample-entrypoint-cmd`.
6. **buildx (multi-architecture images):** You can use a feature called `buildx` to create images for multiple architectures from a single Dockerfile. This video goes into depth on that topic: [https://www.youtube.com/watch?v=hWSHtHasJUI](https://www.youtube.com/watch?v=hWSHtHasJUI).
    - The `make build-multiarch` make target demonstrates using this feature (and the images can be seen here: [https://hub.docker.com/r/sidpalas/multi-arch-test/tags](https://hub.docker.com/r/sidpalas/multi-arch-test/tags).