In this section of the course we will build out a Dockerfile for the NodeJS API, starting with a simple naive approach, and systematically improving it!]
## Emoji Legend:

```bash
🔒 - Security improvement
🏎️ - Build speed improvement
👁️ - Clarity improvement
```

## Naive Implementation

This Dockerfile starts from the official [node container image from DockerHub](https://hub.docker.com/_/node), copies in the entire build context, installs dependencies with npm, and sets a command to be run upon startup.

```bash
FROM node
COPY . .
RUN npm install
CMD [ "node", "index.js" ]
```

While this will technically work, there are **many** ways in which we can improve it!

## Pin the Base Image (🔒+🏎️)

The first way we can improve the Dockerfile is by pinning the base image to a specific version. With no tag, Docker will use the `"latest"` tag which is the default tag applied to images. This would cause the base image to change with each new update to the upstream image, inevitably breaking our application.

We can choose a specific base image that is small and secure to meet the needs of our application.

```bash
#-------------------------------------------
# Pin specific version (use slim for reduced image size)
FROM node:19.6-bullseye-slim
#-------------------------------------------
COPY . .
RUN npm install
CMD [ "node", "index.js" ]
```

Pinning to the minor version should prevent known breaking changes while still allowing patch versions containing bugfixes to be utilized. If we want to truly lock the base image we can refer to a specific image hash such as:

```bash
FROM node:19.6-bullseye-slim@sha256:e684615bdfb71cb676b3d0dfcc538c416f7254697d8f9639bd87255062fd1681
```

## Set a Working Directory (👁️)

By default, our base image has the root path (`/`) as its working directory, but we should set it to something else based on the conventions of our specific language + framework.

This will provide a dedicated place in the filesystem for our app.

```bash
FROM node:19.6-bullseye-slim
#-------------------------------------------
# Specify working directory other than /
WORKDIR /usr/src/app
#-------------------------------------------
COPY . .
RUN npm install
CMD [ "node", "index.js" ]
```

## Copy package.json and package-lock.json Before the Source Code (🏎️)

Each instruction within the Dockerfile creates a new layer within the image. Docker caches these layers to speed up subsequent builds. Previously, every change to the source code would invalidate the layer cache for the `COPY . .` instruction, causing the build to reinstall all of the dependencies (which can be **SLOW!**).

By copying only the dependency configuration files before running `npm install` we can protect the layer cache and avoid reinstalling the dependencies with each source code change.

We can also use a [`.dockerignore`](https://docs.docker.com/engine/reference/builder/#dockerignore-file) file to specify files that should not be included in the container image (such as the `node_modules` directory).

```bash
FROM node:19.6-bullseye-slim
WORKDIR /usr/src/app
#-------------------------------------------
# Copy only files required to install dependencies (better layer caching)
COPY package*.json ./
RUN npm install
# Copy remaining source code AFTER installing dependencies.
# Again, copy only the necessary files
COPY ./src/ .
#-------------------------------------------
CMD [ "node", "index.js" ]
```

## Use a non-root USER (🔒)

If configured properly, containers provide some protection (via user namespaces) between a root user inside a container and the host system user, but setting to a non-root user provides another layer to our [defense in depth](https://csrc.nist.gov/glossary/term/defense_in_depth) security approach!

The node base image already has a user named `node` we can use for this purpose.

```bash
FROM node:19.6-bullseye-slim
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
#-------------------------------------------
# Use non-root user
# Use --chown on COPY commands to set file permissions
USER node
COPY --chown=node:node ./src/ .
#-------------------------------------------
CMD [ "node", "index.js" ]
```

## Configure for the Production Environment (🔒 + 🏎️)

Many Node.js packages look for the `NODE_ENV` environment variable and behave differently if it is set to production (reduced logging, etc...). We can set this within the Dockerfile to ensure it will be set at runtime by default.

Also, rather than using `npm install` it is preferable to use `npm ci` or ["clean install"](https://docs.npmjs.com/cli/v9/commands/npm-ci) which requires the use of a `package-lock.json` file and ensures the installed dependencies match the fully specified versions from that file. By using `--only=production` we can avoid installing unnecessary development dependencies reducing the attack surface area and further reducing the image size.

```bash
FROM node:19.6-bullseye-slim
#-------------------------------------------
# Set NODE_ENV
ENV NODE_ENV production
#-------------------------------------------
WORKDIR /usr/src/app
COPY package*.json ./
#-------------------------------------------
# Install only production dependencies
RUN npm ci --only=production
#-------------------------------------------
USER node
COPY --chown=node:node ./src/ .
CMD [ "node", "index.js" ]
```

## Add Useful Metadata (👁️)

There are a few Dockerfile instructions that don't change the container runtime behavior, but do provide useful metadata for users of the resulting container image.

We can add `LABEL` instructions with various annotations about the container image. For example we might want to include the Dockerfile author, version, licenses, etc... A set of suggested annotation keys from the Open Container Initiative can be found here: [https://github.com/opencontainers/image-spec/blob/main/annotations.md](https://github.com/opencontainers/image-spec/blob/main/annotations.md).

The `EXPOSE` command tells end users the port number that the containerized application expects to listen on. The port will still need to be published at runtime, but it is useful to include this instruction to make it clear to end users which port should be opened.

```bash
FROM node:19.6-bullseye-slim
#-------------------------------------------
# Use LABELS to provide additional info
LABEL org.opencontainers.image.authors="sid@devopsdirective.com"
#-------------------------------------------
ENV NODE_ENV production
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci --only=production
USER node
COPY --chown=node:node ./src/ .
#-------------------------------------------
# Indicate expected port
EXPOSE 3000
#-------------------------------------------
CMD [ "node", "index.js" ]
```

## Use a Cache Mount to Speed Up Dependency Installation (🏎️)

[Buildkit](https://docs.docker.com/build/buildkit/) provides many useful features, including the ability to specify a cache mount for specific `RUN` instructions within a Dockerifle. By specifying a cache in this way, changing a dependency won't require re-downloading all dependencies from the internet, because previously installed dependencies will be stored locally.

**_Note:_** If building the image in a remote continuous Integration system (e.g. GitHub Actions), we would need to configure that system to store and retrieve this cache across pipeline runs.

```bash
FROM node:19.6-bullseye-slim
LABEL org.opencontainers.image.authors="sid@devopsdirective.com"
ENV NODE_ENV production
WORKDIR /usr/src/app
COPY package*.json ./
#-------------------------------------------
# Use cache mount to speed up install of existing dependencies
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm ci --only=production
#-------------------------------------------
USER node
COPY --chown=node:node ./src/ .
EXPOSE 3000
CMD [ "node", "index.js" ]
```

## Use a Multi-Stage Dockerfile (👁️)

[Multi-stage builds](https://docs.docker.com/build/building/multi-stage/) are a docker feature that helps to optimize container images by including multiple independent stages within a Dockerfile.

By splitting out separate development and production image stages we can have an ergonomic dev environment with dev dependencies, hot reloading, etc... but retain security and size improvements for deployment.

Shared steps can be built into a `base` stage and then customizations can be built on top of that base.

```bash
#-------------------------------------------
# Name the first stage "base" to reference later
FROM node:19.6-bullseye-slim AS base
#-------------------------------------------
LABEL org.opencontainers.image.authors="sid@devopsdirective.com"
WORKDIR /usr/src/app
COPY package*.json ./
#-------------------------------------------
# Use the base stage to create dev image
FROM base AS dev
#-------------------------------------------
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm install
COPY . .
CMD ["npm", "run", "dev"]
#-------------------------------------------
# Use the base stage to create separate production image
FROM base AS production
#-------------------------------------------
ENV NODE_ENV production
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm ci --only=production
USER node
COPY --chown=node:node ./src/ .
EXPOSE 3000
CMD [ "node", "index.js" ]
```

## Image Size

As we progressed through these improvements, we reduced the final image size from 1.03GB (😳) to a much more reasonable 250MB (😎)!
![[Pasted image 20240923165153.png]]