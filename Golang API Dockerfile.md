In this section of the course we will build out a Dockerfile for the Golang API, starting with a simple naive approach, and systematically improving it!

## Emoji Legend:

```bash
🔒 - Security improvement
🏎️ - Build speed improvement
👁️ - Clarity improvement
```

## Naive Implementation

This Dockerfile starts from the official [golang container image from DockerHub](https://hub.docker.com/_/golang), sets the working directory, copies in the entire build context, installs dependencies with `go mod`, and sets a command to be run upon startup.

```bash
FROM golang
WORKDIR /app
COPY . .
RUN go mod download
CMD ["go", "run", "./main.go"]
```

While this will technically work, there are **many** ways in which we can improve it.

## Pin the Base Image (🔒+🏎️)

The first way we can improve the Dockerfile is by pinning the base image to a specific version. With no tag, Docker will use the `"latest"` tag which is the default tag applied to images. This would cause the base image to change with each new update to the upstream image, inevitably breaking our application.

We can choose a specific base image that is small and secure to meet the needs of our application.

```bash
#-------------------------------------------
# Pin specific version for stability. Use debian for easier build utilities.
FROM golang:1.19-bullseye AS build
#-------------------------------------------
WORKDIR /app
COPY . .
RUN go mod download
CMD ["go", "run", "./main.go"]
```

Pinning to the minor version should prevent known breaking changes while still allowing patch versions containing bugfixes to be utilized. If we want to truly lock the base image we can refer to a specific image hash such as:

```bash
FROM golang:1.19-bullseye@sha256:1370f30629243bb65e3e0f780ae08a54e50fc5b7e96f0b79e62ee846788d1178
```

## Build the Binary in the Dockerfile (🏎️)

The `go run ./main.go` command both builds and runs the application. By using this as the `CMD` the container will need to build the application upon startup every time.

We should build the application within the container image and then execute the build binary upon startup.

```bash
FROM golang:1.19-bullseye
WORKDIR /app
COPY . .
RUN go mod download
#-------------------------------------------
# Compile application during build rather than at runtime
RUN go build -o api-golang
CMD ["./api-golang"]
#-------------------------------------------
```

## Copy go.mod and go.sum Before Source Code (🏎️)

Each instruction within the Dockerfile creates a new layer within the image. Docker caches these layers to speed up subsequent builds. Previously, every change to the source code would invalidate the layer cache for `COPY . .` causing the build to reinstall all of the dependencies (which can be SLOW!).

By copying only the dependency configuration files before running `go mod download` we can protect the layer cache and avoid reinstalling the dependencies with each source code change.

We can also use a [`.dockerignore` file](https://docs.docker.com/engine/reference/builder/#dockerignore-file) to specify files that should not be included in the container image.

```bash
FROM golang:1.19-bullseye
WORKDIR /app
#-------------------------------------------
# Copy only files required to install dependencies (better layer caching)
COPY go.mod go.sum ./
RUN go mod download
COPY . .
#-------------------------------------------
RUN go build -o api-golang
CMD ["./api-golang"]
```

## Separate Build and Deploy Stages

In order to build the application we need a fair amount of utilities associated with golang (included in the golang base image). However, those are not necessary at runtime.

We can use Docker's [multi-stage build feature](https://docs.docker.com/build/building/multi-stage/) to make our final deployable image MUCH smaller!

```
FROM golang:1.19-bullseye AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
#-------------------------------------------
# Add flags to statically link binary
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o api-golang
# Use separate stage for deployable image
FROM scratch
WORKDIR /
# Copy the binary from the build stage
COPY --from=build /app/api-golang api-golang
#-------------------------------------------
CMD ["/api-golang"]
```

## Set ENV and Expose Port (👁️)

The API framework we are using (Gin) uses the `GIN_MODE` environment variable to determine if it is running in a development or production environment.

We can set `GIN_MODE=release` for our deployable image so it will run in production mode.

```bash
FROM golang:1.19-bullseye AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o api-golang

FROM scratch
WORKDIR /
#-------------------------------------------
# Set gin mode
ENV GIN_MODE=release
#-------------------------------------------
COPY --from=build /app/api-golang api-golang
CMD ["/api-golang"]
```

## Use a non-root USER (🔒)

If configured properly, containers provide some protection (via user namespaces) between a root user inside a container and the host system user, but setting to a non-root user provides another layer to our defense in depth security approach!

We can use the `useradd` command to add a non-root user to the build stage, and then copy the corresponding files into scratch to use it.

```bash
FROM golang:1.19-bullseye AS build
#-------------------------------------------
# Add non root user
RUN useradd -u 1001 nonroot
#-------------------------------------------
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o api-golang

FROM scratch
WORKDIR /
ENV GIN_MODE=release
#-------------------------------------------
# Copy the passwd file
COPY --from=build /etc/passwd /etc/passwd
COPY --from=build /app/api-golang api-golang
# Use nonroot user
USER nonroot
#-------------------------------------------
CMD ["/api-golang"]
```

## Add Useful Metadata (👁️)

There are a few Dockerfile instructions that don't change the container runtime behavior, do provide useful metadata for users of the resulting container image.

We can add `LABEL` instructions with various annotations about the container image. For example we might want to include the Dockerfile author, version, licenses, etc... A set of suggested annotation keys from the Open Container Initiative can be found here: https://github.com/opencontainers/image-spec/blob/main/annotations.md.

The `EXPOSE` command tells end users the port number that the containerized application expects to listen on. The port will still need to be published at runtime, but it is useful to include this instruction to make it clear to end users which port should be opened.

```bash
FROM golang:1.19-bullseye AS build
#-------------------------------------------
# Use LABELS to provide additional info
LABEL org.opencontainers.image.authors="sid@devopsdirective.com"
#-------------------------------------------
RUN useradd -u 1001 nonroot
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o api-golang

FROM scratch
WORKDIR /
ENV GIN_MODE=release
#-------------------------------------------
# Indicate expected port
EXPOSE 8080
#-------------------------------------------
COPY --from=build /etc/passwd /etc/passwd
COPY --from=build /app/api-golang api-golang
USER nonroot
CMD ["/api-golang"]
```

## Use a Cache Mount to Speed Up Dependency Installation (🏎️)

[Buildkit](https://docs.docker.com/build/buildkit/) provides many useful features, including the ability to specify a cache mount for specific `RUN` instructions within a Dockerifle. By specifying a cache in this way, changing a dependency won't require redownloading all dependencies from the internet because previously installed dependencies will be stored locally.

**_Note:_** If building the image in a remote continuous Integration system (e.g. GitHub Actions), we would need to configure that system to store and retrieve this cache across pipeline runs.

```bash
FROM golang:1.19-bullseye AS build
RUN useradd -u 1001 nonroot
WORKDIR /app
COPY go.mod go.sum ./
#-------------------------------------------
# Use cache mount to speed up install of existing dependencies
RUN --mount=type=cache,target=/go/pkg/mod \
  --mount=type=cache,target=/root/.cache/go-build \
  go mod download
#-------------------------------------------
COPY . .
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o api-golang

FROM scratch
ENV GIN_MODE=release
EXPOSE 8080
WORKDIR /
COPY --from=build /etc/passwd /etc/passwd
COPY --from=build /app/api-golang api-golang
USER nonroot
CMD ["/api-golang"]
```

## Image Size

As we progressed through these improvements, we reduced the final image size from 926MB (😳) to a just 17MB (😎)!
![[Pasted image 20240923165426.png]]