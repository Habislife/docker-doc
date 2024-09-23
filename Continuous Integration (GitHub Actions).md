Continuous integration is the idea of executing some actions (for example build, test, etc...) automatically as you push code to your version control system.

For containers, there are a number of things we may want to do:

1. Build the container images
2. Execute tests
3. Scan container images for vulnerabilities
4. Tag images with useful metadata
5. Push to a container registry

## GitHub Actions

GitHub Actions is a continuous integration pipeline system built into GitHub.

You add configuration files to `.github/workflows` within the repo and GitHub will automatically execute them based on the conditions you set!

GitHub actions has a [public marketplace](https://github.com/marketplace?type=actions) where people can publish open source actions that help make the process of writing your pipelines easier and faster. We will use a number of these actions as we build out a workflow for our repo.

**Note:** The workflow file shown in the course can be found at [https://github.com/sidpalas/devops-directive-docker-course/blob/main/11-development-workflow/docker-compose-dev.yml](https://github.com/sidpalas/devops-directive-docker-course/blob/main/11-development-workflow/docker-compose-dev.yml)

### Execution conditions

Common events used to trigger workflows include:

- Push events to one or more branches (e.g. with each update to the `main` branch)
- Creation of tags (e.g. tag that matches `v*` pattern indicating a release)
- Pull request creation/modification (usually to execute tests)

In this case we want to run our workflow on push events to the `github-action` branch and on any `v*` tags. To specify this we use the following yaml:

```yaml
on:
  push:
    branches:
      - "github-action"
    tags:
      - "v*"
```

### Build, Tag, and Push

We can then specify one or more jobs. To keep things simple for the course I included a single job that will build one of our container images, tag it, push it to Dockerhub, and scan it for vulnerabilities.

The job is given a name and a specific machine type to run on.

```yaml
jobs:
  build-tag-push:
    runs-on: ubuntu-latest
    steps:
      - ...
```

We then proceed through the following steps:

1. Check out the code:

A standard action which checks out the code from the repo at the relevant commit.

```yaml
- name: Checkout
  uses: actions/checkout@v3
```

2. Generate image tags:

Uses an action from Docker to generate useful tags based on information about the triggering event, the commit sha, and the current timestamp.

```yaml
- name: Docker meta
  id: meta
  uses: docker/metadata-action@v4
  with:
    images: |
      sidpalas/devops-directive-docker-course-api-node
    tags: |
      type=raw,value=latest
      type=ref,event=branch
      type=ref,event=pr
      type=semver,pattern={{version}}
      type=semver,pattern={{major}}.{{minor}}
      type=raw,value={{date 'YYYYMMDD'}}-{{sha}}
```

3. Login to DockerHub

Uses an action from Docker + secrets stored in the repo to authenticate to Dockerhub.

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

4. Build and push image

Use an action from Docker along with the output of the tag generation step to build and push the container image.

```yaml
- name: Build and push
  uses: docker/build-push-action@v4
  with:
    file: ./06-building-container-images/api-node/Dockerfile.8
    context: ./05-example-web-application/api-node/
    push: true
    tags: ${{ steps.meta.outputs.tags }}
```

5. Scan image for vulnerabilities

Use an action from Trivy to run their security scanner against the built image and fail if any `CRITICAL` level vulnerabilities are found.

```yaml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: "sidpalas/devops-directive-docker-course-api-node:latest"
    format: "table"
    exit-code: "1"
    ignore-unfixed: true
    vuln-type: "os,library"
    severity: "CRITICAL"
```

## Additional Resources

For more examples and advanced use cases of GitHub Actions and Docker CI/CD, check out Brett Fisher's [Docker CI/CD Automation repository](https://github.com/BretFisher/docker-ci-automation).

You can also watch his talk on the subject for a full walkthrough: [https://www.youtube.com/watch?v=aZzV6X7XhyI](https://www.youtube.com/watch?v=aZzV6X7XhyI).