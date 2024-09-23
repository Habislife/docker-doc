In this lesson we craft the `docker run` commands to run our containerized application.

## 1. Building the Docker Images

First, we need to build our container images.

Because of the repo structure, we need to pass the `-f` flag to tell docker where our Dockerfiles live, and pass a context pointing to the correct subdirectory:

```bash
DOCKERCONTEXT_DIR:=../05-example-web-application/
DOCKERFILE_DIR:=../06-building-container-images/

docker build -t client-react-vite -f ${DOCKERFILE_DIR}/client-react/Dockerfile.3 ${DOCKERCONTEXT_DIR}/client-react/
docker build -t client-react-ngnix -f ${DOCKERFILE_DIR}/client-react/Dockerfile.5 ${DOCKERCONTEXT_DIR}/client-react/
docker build -t api-node -f ${DOCKERFILE_DIR}/api-node/Dockerfile.7 ${DOCKERCONTEXT_DIR}/api-node/
docker build -t api-golang -f ${DOCKERFILE_DIR}/api-golang/Dockerfile.6 ${DOCKERCONTEXT_DIR}/api-golang/
```

## 2. Create a Docker Network

Create a new Docker network called my-network:

```bash
docker network create my-network
```

Check that the network was created:

```bash
docker network ls
```

## 3. Run the Postgres Container

We will work our way forward, starting with the database and ending with our react client.

```bash
docker run -d \
  --name db \
  --network my-network \
  -e POSTGRES_PASSWORD=foobarbaz \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  --restart unless-stopped \
  postgres:15.1-alpine
```

We have seen commands like this to start a PostgeSQL container, but now we are adding:

- `-d`: to run it in the background
- `--network my-network`: to attach it to the network we created
- `--restart unless-stopped`: to restart the DB if it crashes

## 4. Run the Node API Container

```bash
DATABASE_URL:=postgres://postgres:foobarbaz@db:5432/postgres
docker run -d \
  --name api-node \
  --network my-network \
  -e DATABASE_URL=${DATABASE_URL} \
  -p 3000:3000 \
  --restart unless-stopped \
  api-node
```

One option worth calling out is:

- `-e DATABASE_URL=${DATABASE_URL}`: which we use to pass the database credentials to the application

## 5. Run the Golang API Container

```bash
DATABASE_URL:=postgres://postgres:foobarbaz@db:5432/postgres
docker run -d \
  --name api-golang \
  --network my-network \
  -e DATABASE_URL=${DATABASE_URL} \
  -p 8080:8080 \
  --restart unless-stopped \
  api-golang
```

You will notice this is nearly identical to the `api-node` command except it uses a different container image and publishes a different port.

## 6. Run the Vite and Nginx Clients

We will run both the development server with `vite` as well as the more production ready `nginx` based client:

```bash
docker run -d \
		--name client-react-vite \
		--network my-network \
		-v ${PWD}/client-react/vite.config.js:/usr/src/app/vite.config.js \
		-p 5173:5173 \
		--restart unless-stopped \
		client-react-vite

docker run -d \
		--name client-react-nginx \
		--network my-network \
		-p 80:8080 \
		--restart unless-stopped \
		client-react-ngnix
```

The options worth noting are:

- `-v ${PWD}/client-react/vite.config.js:/usr/src/app/vite.config.js`: which mounts a new vite config file in at runtime
- `-p 80:8080`: to publish port 8080 in the container to port 80 on our localhost

## Cleaning Up

To stop and remove all of these containers we can use:

```bash
docker stop db api-node api-golang client-react-vite client-react-nginx

docker rm db api-node api-golang client-react-vite client-react-nginx

docker network rm my-network
```