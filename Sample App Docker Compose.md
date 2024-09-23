Docker Compose offers a simpler and more efficient way to manage multiple containers compared to using individual `docker run` commands.

Now we will adapt our run commands from the previous lesson into a compose file!

One nice resource to help with this is the `composerizer` tool from Mark Larah [https://github.com/magicmark/composerize](https://github.com/magicmark/composerize)

## Top level structure

To use docker compose, we write a yaml file that contains all of the necessary configuration options.

For each `docker run` option there is a corresponding compose yaml syntax.

```yaml
version: "3.8"
services:
  service-a:
    image: foo
    # configuration options for service-a
  service-b:
    image: bar
    # configuration options for service-b
```

Within the `yaml` file we specify which version of the docker compose syntax we are using, and then create a block for each service.

The full `docker-compose.yml` file can be found in the GitHub repo here: [https://github.com/sidpalas/devops-directive-docker-course/blob/main/08-running-containers/docker-compose.yml](https://github.com/sidpalas/devops-directive-docker-course/blob/main/08-running-containers/docker-compose.yml)

## Postgres Container

```yaml
  db:
    image: postgres:15.1-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=foobarbaz
    networks:
      - backend
    ports:
      - 5432:5432
    restart: unless-stopped
volumes:
  pgdata:
networks:
  backend:
```

Notice how we specify the named volume and network in a separate section from the service. This is because the volumes and networks are managed separately from the containers.

## API Services

```yaml
  api-node:
    image: api-node
    build:
      context: ../05-example-web-application/api-node/
      dockerfile: ../../06-building-container-images/api-node/Dockerfile.7
    init: true
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://postgres:foobarbaz@db:5432/postgres
    networks:
      - frontend
      - backend
    ports:
      - 3000:3000
    restart: unless-stopped
  api-golang:
    image: api-golang
    build:
      context: ../05-example-web-application/api-golang/
      dockerfile: ../../06-building-container-images/api-golang/Dockerfile.6
    init: true
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://postgres:foobarbaz@db:5432/postgres
    networks:
      - frontend
      - backend
    ports:
      - 8080:8080
    restart: unless-stopped
networks:
  frontend:
  backend:
```

In addition to the direct mapping of the run command options, there are a few items worth mentioning:

- I have added two docker networks `frontend` and `backend` This allows for further isolation than the single docker network we were using previously because the client application will only be attached to the `frontend` network and therefore wont be able to communicate directly with the database.
- The `depends_on: db` option tells docker to wait until the `db` container starts before starting these containers. (Note: it does not wait for the database to be ready, just that the container has started.)
- The `build` section allows you to specify a `dockerfile` and `context` to enable compose to manage building the container images.

## Front End Containers

```yaml
client-react-vite:
    image: client-react-vite
    build:
      context: ../05-example-web-application/client-react/
      dockerfile: ../../06-building-container-images/client-react/Dockerfile.3
    init: true
    volumes:
      - ./client-react/vite.config.js:/usr/src/app/vite.config.js
    networks:
      - frontend
    ports:
      - 5173:5173
  client-react-nginx:
    labels:
      shipyard.primary-route: true
      shipyard.route: '/'
    image: client-react-nginx
    build:
      context: ../05-example-web-application/client-react/
      dockerfile: ../../06-building-container-images/client-react/Dockerfile.5
    init: true
    networks:
      - frontend
    ports:
      - 80:8080
    restart: unless-stopped
```

Nothing particular special going on here. These are a direct translation of the options we used before.

## Using Docker Compose

With our `docker-compose.yml` defined we can now:

1. Build our container images:

```bash
docker compose build
```

2. Run our application:

```bash
docker compose up

# or run in the background with:
docker compose up -d
```

3. Stop our application:

```bash
# Press CTRL + C if running in the foreground

# If running in the background (-d)
docker compose stop
```

As you can see, defining all of the runtime options in the `docker-compose.yml` file is much easier to understand and work with than managing lots of separate `docker run` commmands!