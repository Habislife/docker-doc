We discuss what a container is, the difference between a container and a container image, and the Open Container Initiative (OCI) that standardizes container formats.
## Container and Container Image
A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application. This package contains:

- Underlying OS dependencies
- Runtime dependencies (e.g., Python runtime)
- Libraries (e.g., SQL Alchemy, FastAPI)
- Application code

A container image is like a class in object-oriented programming, while a container is an instantiation of that class. A container allows us to create one or more standardized copies that are the same every time.
![[Pasted image 20240923153720.png]]

## Open Container Initiative (OCI)

The OCI is an industry collaboration that aims to create open standards for container formats. It was founded by companies like Docker, Google, VMware, Microsoft, Dell, IBM, and Oracle. The OCI defines three primary specifications:

1. **Image Specification:** Defines the image's metadata and format, including a serializable file system.
2. **Runtime Specification:** Describes how to run a container using an image adhering to the Image Specification.
3. **Distribution Specification:** Outlines how images should be distributed, such as through registries, pushing, and pulling images.

Docker is a specific implementation of the OCI standard. When referring to Docker images or Docker container images, it means the Docker implementation of the OCI specification.![[Pasted image 20240923153752.png]]