# Deploying with Docker Swarm

The general approach we will use to deploy our application is:

## 1. Create a virtual machine

Create a virtual machine on your preferred cloud provider. I use Civo in the video, but you are welcome to choose another (e.g., AWS, GCP, or Azure).

## 2. Set up the firewall

Configure the firewall for your virtual machine to allow inbound traffic on the necessary ports.

## 3. Install Docker Engine

Install Docker Engine on your virtual machine using the script located at [get.docker.com](https://get.docker.com/).

```bash
ssh ubuntu@YOUR_IP_ADDRESS

curl https://get.docker.com/ | sh
sudo groupadd docker
sudo usermod -aG docker $USER
```

## 4. Initialize Docker Swarm

Connect your local Docker client to the Docker Engine running on the virtual machine using the `DOCKER_HOST` environment variable. Initialize Docker Swarm on the remote machine.

```bash
export DOCKER_HOST:="ssh://ubuntu@YOUR_IP_ADDRESS"
docker swarm init
```

## 5. Modify Docker Compose file

Add health checks to your Docker Compose file to allow Swarm to periodically ping your service and restart it if necessary.

Add a deployment strategy to enable zero-downtime deployments.

## 6. Add support for secrets

Implement Docker secrets in your Docker Compose file to securely handle credentials for your database.

We can use the following command to create the secrets in the swarm cluster:

```bash
export DOCKER_HOST:="ssh://ubuntu@YOUR_IP_ADDRESS"
printf "foobarbaz" | DOCKER_HOST=${DOCKER_HOST} docker secret create postgres-passwd -
printf "postgres://postgres:foobarbaz@db:5432/postgres" | DOCKER_HOST=${DOCKER_HOST} docker secret create database-url -
```

## 7. Build and push container images

Build your container images and push them to Docker Hub with specific image tags.

## 8. Deploy the stack

Deploy your stack onto the virtual machine and see it running.

```bash
export DOCKER_HOST:="ssh://ubuntu@YOUR_IP_ADDRESS"
docker stack deploy -c docker-swarm.yml example-app
```

## Disclaimer

This guide demonstrates deploying the database alongside the applications. While this is acceptable, you should also consider implementing backups and/or using a Database-as-a-Service for production applications with important user data.