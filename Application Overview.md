To demonstrate how actual companies and organizations use Docker and Containers, I created a minimal 3-tier web application.

It has the following components:

- React front end
- NodeJS API
- Golang API
- Postgres Database

Each component is very minimal, but allows us to highlight the important aspects of containerizing and configuring the system.

Before we containerize it though, first we can explore the code and run it on our host system directly.
## 1. Setting up the Postgres Database

First, we'll run the Postgres database using the Docker run command. We will pass a `POSTGRES_PASSWORD` environment variable, use a volume mount to persist the data outside of the container, and publish port 5432 (the default port that Postgres runs on).

```bash
docker run -e POSTGRES_PASSWORD=foobarbaz -v pgdata:/var/lib/postgresql/data -p 5432:5432 -d postgres:15.1-alpine
```

## 2. Running the Node.js API

The Node.js API is located within the `api-node` subdirectory. First, ensure that you are using the correct version of Node (19.4 in this example). To install the dependencies, run `npm install` in the `api-node` directory.

To run the application, execute the following command:

```bash
DATABASE_URL=postgres://postgres:foobarbaz@localhost:5432/postgres npm run dev
```

This will start the API server, listening on port 3000. The API returns the current timestamp and an API key in the JSON response.

## 3. Running the Golang API

The Golang API is located within the `api-golang` subdirectory. First, set the `GOPATH` environment variable to the appropriate workspace directory. To install the dependencies, run `go mod download` in the `api-golang` directory.

To run the application, execute the following command:

```bash
DATABASE_URL=postgres://postgres:foobarbaz@localhost:5432/postgres go run main.go
```

This will start the API server, listening on port 8080. The API returns a similar response to the Node.js API, with the current timestamp and an API key in the JSON response.

## 4. Running the React Client

The React client is located within the `client-react` subdirectory. First, ensure that you are using the correct version of Node (19.4 in this example). To install the dependencies, run `npm install` in the `client-react` directory.

To run the application, execute the following command:

```bash
npm run dev
```

This will start the development server, listening on port 5173. The React app calls both the Node.js and Golang APIs and displays the responses.