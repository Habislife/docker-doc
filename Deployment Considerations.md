When deploying containers there are a number of important things to consider:

- Security
- Developer experience (ergonomics)
- Scalability
- Persistent storage
- Cost

## Deployment Approach
![[Pasted image 20240923172150.png]]

In this module of the course we will deploy our application using a single node Docker Swarm cluster.

The repo also contains configurations for deploying to railway.app and Kubernetes. Those configurations are covered in separate bonus videos that can be found at [https://links.devopsdirective.com/docker-gumroad](https://links.devopsdirective.com/docker-gumroad)

## Why not Docker Compose?

Docker Compose has some limitations for production workloads:

- No zero-downtime deployment support
- No easy rollback mechanism
- No native support for secrets to manage credentials
- Limited to single host deployment

Docker Swarm addresses these limitations, making it a better choice for production deployments.