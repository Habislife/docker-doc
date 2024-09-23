In this guide, we'll explore ephemeral environments, which are short-lived, isolated environments for testing, validation, and QA.

We'll be using the Shipyard platform [https://shipyard.build](https://shipyard.build/) to easily set up ephemeral environments for our sample application.

## Setting Up Shipyard Account

1. Visit the [shipyard.build](https://shipyard.build/) website and click `Log In`
    - Shipyard has also provided an exclusive coupon code for students of this course! The first 300 people to use the code **"DEVOPSDIRECTIVE"** during signup will get an additional 30 days free on either their startup or business tier plans!
    - Thank you to Shipyard for sponsoring the course video and supporting the container community! 🙏
2. Log in with your GitHub account and authorize Shipyard to access your account.
3. Connect your GitHub organization and grant Shipyard the necessary permissions to access your repositories.

## Configuring Application

1. Click the `+ Application` button
2. Select the repository and the base environment branch (e.g., `main`)
3. Choose the appropriate Docker Compose file that contains all the services you need
4. Add the necessary labels to your Docker Compose file so Shipyard can properly route traffic to your services (see the code snippet below).

## Updating the Docker Compose File

To help Shipyard detect and utilize routes, add a labels field to each of service in the Docker Compose file:

```yaml
services:
  react-nginx:
    labels:
      shipyard.primary-route: true
      shipyard.route: "/"
    # ...
  api-node:
    labels:
      shipyard.route: "/api/node/"
      shipyard.route.rewrite: true
    # ...
  api-golang:
    labels:
      shipyard.route: "/api/golang/"
      shipyard.route.rewrite: true
    # ...
```

After pushing these changes to the branch selected above, Shipyard will automatically build your container images and deploy a copy of the application.

## Monitoring and Accessing Ephemeral Environments
1. Visit the Shipyard dashboard to see the build and deployment progress.
2. Once the environment is ready, you can visit it by logging in using GitHub or Google OAuth.
3. To grant access to others, add their GitHub or Google usernames in the Visitors tab.

## Shipyard CLI

1. Install the Shipyard CLI by following the instructions in their documentation.
2. Obtain your Shipyard API token by asking the Shipyard team to enable the CLI for your account. Your API token will then be available in your profile settings.
3. Use the CLI commands to manage your environments, view logs, port forward, and more.

## Integrating Shipyard with GitHub

1. Enable GitHub commit checks and PR comments in the Shipyard application settings.
2. Set conditions for deploying on specific pull requests, such as labels or branch name patterns.
3. Shipyard will add comments to your pull requests with links to the built environments and build details.