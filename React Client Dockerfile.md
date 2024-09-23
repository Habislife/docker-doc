In this section of the course we will build out a Dockerfile for the React Client API, starting with a simple naive approach, and systematically improving it!

## Emoji Legend:

```bash
🔒 - Security improvement
🏎️ - Build speed improvement
👁️ - Clarity improvement
```

The Dockerfile for our React based front end is kind of a hybrid between the NodeJS api and the Golang api Dockerfiles. We will use Node + NPM at first, but then build our site as static HTML, CSS, and JS files that we will serve using a separate deployable stage.

## Naive Implementation
The naive implementation should look very familiar, since it't nearly identical to that of the Node API.

Running a container from this image will run our `vite` development server.

```bash
FROM node
COPY . .
RUN npm install
CMD ["npm", "run", "dev"]
```

## Pin the Base Image (🔒+🏎️)

As always, we want to use a specific version of our base image to avoid nasty surprises when the upstream `latest` tag changes.

```bash
#-------------------------------------------
# Pin specific version for stability
FROM node:19.4-bullseye
#-------------------------------------------
COPY . .
RUN npm install
CMD ["npm", "run", "dev"]
```

## Set WORKDIR and COPY package.json File

We want to specify a working directory other than `/` and separate out the `COPY` commands for our `package.json` and `package-lock.json` files to improve caching.

```bash
FROM node:19.4-bullseye
#-------------------------------------------
# Specify working directory other than /
WORKDIR /usr/src/app
# Copy only files required to install dependencies (better layer caching)
COPY package*.json ./
#-------------------------------------------
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

## Use a Cache Mount (🏎️)

To help speed up dependency installation we can add a cache mount and tell npm to use it.

```bash
FROM node:19.4-bullseye
WORKDIR /usr/src/app
COPY package*.json ./
#-------------------------------------------
# Use cache mount to speed up install of existing dependencies
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm install
#-------------------------------------------
COPY . .
CMD ["npm", "run", "dev"]
```

## Separate Build and Deploy Stages

Our React app will be built into a set of static files (HTML, CSS, and JS) that we can then deploy in a variety of ways.

Here we will build the application in one stage and then copy those into a second stage running Nginx!

```bash
FROM node:19.4-bullseye AS build
WORKDIR /usr/src/app
COPY package*.json ./
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm install
COPY . .
RUN npm run build
#-------------------------------------------
# Use separate stage for deployable image
FROM nginxinc/nginx-unprivileged:1.23-alpine-perl
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build usr/src/app/dist/ /usr/share/nginx/html
EXPOSE 8080
#-------------------------------------------
```

## Use COPY --link

One final tweak we can make is to use the `COPY --link` syntax in the second stage. This will allow us to avoid invalidating the layer cache if we change the second stage base image.

```bash
FROM node:19.4-bullseye AS build
WORKDIR /usr/src/app
COPY package*.json ./
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm install
COPY . .
RUN npm run build

FROM nginxinc/nginx-unprivileged:1.23-alpine-perl
#-------------------------------------------
# Use COPY --link to avoid breaking cache if we change the second stage base image
COPY --link nginx.conf /etc/nginx/conf.d/default.conf
COPY --link --from=build usr/src/app/dist/ /usr/share/nginx/html
#-------------------------------------------
EXPOSE 8080
```

## Image Size

As we progressed through these improvements, we reduced the final image size from 1.16GB (😳) to a much more reasonable 77MB (😎)!
![[Pasted image 20240923165644.png]]