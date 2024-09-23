## What is a Dockerfile?

A Dockerfile is a text document that contains all the commands required to create and assemble a container image. It serves as a recipe for your application, starting with a base layer like an operating system, installing language runtime and dependencies, setting up the execution environment, and finally running a command to start your application.

```
👨‍🍳 Application Recipe:
---------------------------------------
1. Start with an Operating System
2. Install the language runtime
3. Install any application dependencies
4. Set up the execution environment
5. Run the application
```
## The Docker Build Context

The Dockerfile is paired with a build context, which is usually a folder or directory on your local system containing your source code. The build context can also be a URL, such as a public GitHub repository. Docker uses the Dockerfile and build context together when running the docker build command to produce a container image.

## The .dockerignore File

A `.dockerignore` file can be included in your build context to tell Docker to ignore certain files. For example, if you've installed `node_modules` locally, you wouldn't want to copy those into the container image, as they will be installed within the Dockerfile. This can prevent incompatibilities between installations on your host system and within the container.
![[Pasted image 20240923164809.png]]
## Writing a Dockerfile

When writing a Dockerfile, you can refer to the [Docker documentation](https://docs.docker.com/engine/reference/builder/) for a list of valid commands. The format for a Dockerfile is relatively simple:

A hash (`#`) is used for comments. Instructions are written in all caps, followed by arguments. For example:

```bash
# This step installs dependencies
RUN apt-get update && apt-get install -y <package_name>
```

## Common Commands in a Dockerfile

Here are some you'll encounter in almost every Dockerfile:

```
FROM: Specifies the base layer or operating system for the container image.
RUN: Executes a command during the build phase.
COPY: Copies files from the build context (e.g., your local system) to the container image.
CMD: Provides a command to be executed when the container starts.
```

## Docker build command

We can take a Dockerfile and a build context and use the `docker build` command to create a Docker Container Image!

```bash
docker build -f Dockerfile .
```

Here the `.` indicates that the current directory should be used as the build context.

**Note:** You only have to pass a Dockerfile name if it is named something other than "Dockerfile"