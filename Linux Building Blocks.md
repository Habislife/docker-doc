Containers leverage three linux features to work their magic:

- cgroups
- namespaces
- union fileystems

Cgroups and namespaces to provide resource constraints and application isolation respectively. They also use an union filesystem that enables images to be built upon common layers, making building and sharing images fast and efficient.

**_Note:_** Docker did not invent containers. For example, LXC containers (https://linuxcontainers.org/) was implemented in 2008, five years before Docker launched. That being said, Docker made huge strides in developer experience, which helped container technologies gain mass adoption and remains one the most popular containerization platforms.

## Cgroups

[Cgroups](https://man7.org/linux/man-pages/man7/namespaces.7.html) are a Linux kernel feature which allow processes to be organized into hierarchical groups whose usage of various types of resources can then be limited and monitored.

With cgroups, a container runtime is able to specify that a container should be able to use (for example):

- Use up to XX% of CPU cycles (cpu.shares)
- Use up to YY MB Memory (memory.limit_in_bytes)
- Throttle reads to ZZ MB/s (blkio.throttle.read_bps_device)

## Namespaces

A [namespace](https://man7.org/linux/man-pages/man7/namespaces.7.html) wraps a global system resource in an abstraction that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource.

Changes to the global resource are visible to other processes that are members of the namespace, but are invisible to other processes.

With namespaces, a container runtime is able to keep processes outside of the container invisible within the container or map the user inside the container to a different user on the host (among other things).

## Union filesystems

A [union filesystem](https://martinheinz.dev/blog/44) allows files and directories of separate file systems, known as branches, to be transparently overlaid, forming a single coherent file system.

Contents of directories which have the same path within the merged branches will be seen together in a single merged directory, within the new, virtual filesystem.

This approach allows for efficient use of space because common layers can be shared. For example, if multiple containers from the same image are created on a single host, the container runtime only has to allocate a thin overlay specific to each container, while the underlying image layers can be shared. More detail on understanding the implications of these filesystem on data persistence can be found in 04-using-3rd-party-containers.
![[Pasted image 20240923155130.png]]
Docker provides a convenient user experience by packaging the underlying technologies (namespaces, cgroups, and union file systems) into an easy-to-use desktop application.


