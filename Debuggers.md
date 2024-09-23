In order to make developing with containers competitive with developing locally, we need the ability to run and attach to debuggers inside the container.

## Aside: Multiple Docker Compose Files

Docker compose has a nice feature that allows you to specify multiple docker compose files at runtime. This way you can define a base configuration and a much smaller overlay with customizations.

For example, lets say we had `docker-compose-a.yml`:

```yaml
services:
  my-service:
    image: foobar
```

and `docker-compose-b.yml`:

```yaml
services:
  my-service:
    command: ["echo", "new command!"]
```

we can then issue the following `docker compose` command to run `my-service` with the updated command:

```bash
docker compose -f docker-compose-a.yml -f docker-compose-b.yml run my-service
```

## Debuggers

As much as we all love sprinkling `console.log()` and `print()` statements, we need to be able to use real debuggers that enable us to set breakpoints, examine the runtime state of our code, etc...

### Node API

NodeJS has a built in debugger we can activate using the `--inspect` flag. We can set up an NPM script to utilize this:

```
    "debug-docker": "nodemon --inspect=0.0.0.0:9229 ./src/index.js",
```

By default, `inspect` would only accept connections from localhost, but in this case we want to accept connections from outside of the container which is why we specify `0.0.0.0` (any host).

Now we can craft a compose overlay `docker-compose-debug.yml` to use this npm script and publish port 9229.

```yaml
services:
  api-node:
    command:
      - "npm"
      - "run"
      - "debug-docker"
    ports:
      - "3000:3000"
      # inspect debug port
      - "9229:9229"
```

With this configuration we can connect to the debugger listening on port 9229.

### Golang API

In the previous lesson we added the following to our Dockerfile to install delve ([https://github.com/go-delve/delve](https://github.com/go-delve/delve)), a golang debugger.

```
RUN go install github.com/go-delve/delve/cmd/dlv@latest
```

Combining this with a compose overlay:

```
services:
  api-golang:
    command:
      - "dlv"
      - "debug"
      - "/app/main.go"
      - "--listen=:4000"
      - "--headless=true"
      - "--log=true"
      - "--log-output=debugger,debuglineerr,gdbwire,lldbout,rpc"
      - "--accept-multiclient"
      - "--continue"
      - "--api-version=2"
    ports:
      - "8080:8080"
      # delve debug port
      - "4000:4000"
```

We can run our remote debugger and connect to it on port 4000.

## Configuring VSCode to use Debuggers

In VSCode, create a launch.json file inside the .vscode folder in your project root directory. Add the following configurations:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Docker: Attach to Node",
      "type": "node",
      "request": "attach",
      "localRoot": "${workspaceFolder}/docker-course/devops-directive-docker-course/05-example-web-application/api-node",
      "remoteRoot": "/usr/src/app",
      "port": 9229
    },
    {
      "name": "Docker: Attach to Golang",
      "type": "go",
      "debugAdapter": "dlv-dap",
      "mode": "remote",
      "request": "attach",
      "port": 4000,
      "remotePath": "/app",
      "substitutePath": [
        {
          "from": "${workspaceFolder}/docker-course/devops-directive-docker-course/05-example-web-application/api-golang",
          "to": "/app"
        }
      ]
    }
  ]
}
```

You may need to adjust the `localRoot`/`remoteRoot` and `substitutePath` settings to match your workspace configuration, but once you do you will be able to attach to the debugger from VSCode.

## Running in debug mode

To use the updated configurations, run docker compose up with the dev and debug configurations together:

```bash
docker-compose -f docker-compose-dev.yml -f docker-compose-debug.yml up --build
```

As discussed earlier, this command will interleave the configuration from both the `docker-compose-dev.yml` and `docker-compose-debug.yml` files.