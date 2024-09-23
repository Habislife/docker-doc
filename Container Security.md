Learn best practices for container image and container runtime security.

There are two main aspects to consider when thinking about container security:

- **Container Image Security:** What vulnerabilities exist in your image that an attacker could exploit?
- **Container Runtime Security:** If an attacker successfully compromises a container, what can they do? How difficult will it be to move laterally?
## Image Security

1. Keep the attack surface area small
    - Use minimal base images to reduce the number of installed components, decreasing the chance of vulnerabilities. ChainGuard is a good source for secure base images.
2. Don't include unnecessary components
    - Exclude components that aren't needed at production time in your Dockerfile.
3. Utilize multi-stage builds
    - Use multi-stage builds for components needed during the build process but not at runtime.
4. Scan images for vulnerabilities
    - Use tools such as Snyk (built into Docker) or Trivy (from Aqua Security) to scan images for potential vulnerabilities.
5. Avoid running as root user
    - Run containers with a Linux user with minimal permissions.
6. Don't store sensitive information in images
    - Treat images as if they're public and inject sensitive data at runtime.
7. Sign images cryptographically
    - Sign your images to prove their origin and ensure their integrity.
8. Pin base images
    - Pin your base images to at least the minor version number to automatically incorporate bug fixes but avoid breaking changes.

## Runtime Security

1. Use user namespace remap
    - Enable the user namespace remap option in Docker daemon (dockerd) to separate container user namespaces from the host system.
2. Set the file system as read-only
    - Configure the file system as read-only for containers that don't require write access.
3. Remove and add capabilities as needed
    - Use the `cap_drop` option to remove all capabilities, then add back any necessary ones.
4. Limit CPU and memory
    - Restrict CPU and memory usage to prevent denial-of-service situations.
5. Set `seccomp` or `AppArmor` profiles
    - Utilize security options to set `seccomp` or `AppArmor` profiles for additional security layers.