Even with layer caching, we don't want to have to rebuild our container image with every code change.

Instead, we want the state of our application in the container to reflect changes immediately. We can achieve this through a combination of bind mounts and hot reloading utilities!

The full `docker-compose.yml` file can be found here: [https://github.com/sidpalas/devops-directive-docker-course/blob/main/11-development-workflow/docker-compose-dev.yml](https://github.com/sidpalas/devops-directive-docker-course/blob/main/11-development-workflow/docker-compose-dev.yml)

## React Frontend

Vite is already designed to handle hot reloading, so all we need to do is bind mount our source code directory into the container at runtime.

```yaml
volumes:
  - type: bind
    source: ../05-example-web-application/client-react/
    target: /usr/src/app/
  - type: volume
    target: /usr/src/app/node_modules
```

Here we can see the source code location is mounted to the path we specified as `WORKDIR` in the `Dockerfile`.

We also add a second volume with NO source mounted to the location of the `node_modules` directory just in case we have installed the node modules locally. This takes precedence over the bind mount and prevents those unwanted files from being mounted in.

## Node API

We will be using `nodemon` to watch for changes and restart the node server.

First, install it as a development dependency:

```bash
npm install --save-dev nodemon
```

Then, refactor the Dockerfile to have a separate `dev` stage which includes development dependencies:

```bash
FROM node:19.6-bullseye-slim AS base
WORKDIR /usr/src/app
COPY package*.json ./

#------------------------------------------------
# Separate dev stage with nodemon and different CMD
FROM base as dev
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm install
COPY . .
# "npm run dev" corresponds to "nodemon src/index.js"
CMD ["npm", "run", "dev"]
#------------------------------------------------

FROM base as production
ENV NODE_ENV production
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm ci --only=production
USER node
COPY --chown=node:node ./src/ .
EXPOSE 3000
CMD [ "node", "index.js" ]
```

We then can specify the `dev` as the target for our docker compose build configuration and add the bind mount to our compose file:

```yaml
build:
  context: ../05-example-web-application/api-node/
  dockerfile: ../../06-building-container-images/api-node/Dockerfile.9
  target: dev
volumes:
  - type: bind
    source: ../05-example-web-application/api-node/
    target: /usr/src/app/
  - type: volume
    target: /usr/src/app/node_modules
```

And our Node API will have hot reloading!

## Golang API

Since golang is a compiled language, for hot reloading, we actually need to recompile with each code change. We will be using [https://github.com/cosmtrek/air](https://github.com/cosmtrek/air) for this.

There is no concept of dev vs. production dependencies within the `go.mod` file, so we can update our Dockerfile to create a separate dev stage and install Air there:

```
FROM golang:1.19-bullseye AS build-base
WORKDIR /app
COPY go.mod go.sum ./
RUN --mount=type=cache,target=/go/pkg/mod \
  --mount=type=cache,target=/root/.cache/go-build \
  go mod download

#------------------------------------------------
FROM build-base AS dev
# Install air for hot reload & delve for debugging
RUN go install github.com/cosmtrek/air@latest && \
  go install github.com/go-delve/delve/cmd/dlv@latest
COPY . .
CMD ["air", "-c", ".air.toml"]
#------------------------------------------------

FROM build-base AS build-production
RUN useradd -u 1001 nonroot
COPY . .
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o api-golang

FROM scratch
ENV GIN_MODE=release
WORKDIR /
COPY --from=build-production /etc/passwd /etc/passwd
COPY --from=build-production /app/healthcheck/healthcheck healthcheck
COPY --from=build-production /app/api-golang api-golang
USER nonroot
EXPOSE 8080
CMD ["/api-golang"]
```

With this updated, we can modify the docker compose to target the dev stage and bind mount the source code:

```yaml
build:
  context: ../05-example-web-application/api-golang/
  dockerfile: ../../06-building-container-images/api-golang/Dockerfile.8
  target: dev
volumes:
  - type: bind
    source: ../05-example-web-application/api-golang/
    target: /app/
```