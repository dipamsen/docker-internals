# Docker Internals

## Outline

- What is a Container?
- What are cgroups and namespaces, how can they be used to build containers
- The Docker ecosystem - containerd, runc, many other tools that are a part of the larger cloud native ecosystem - what they do and how they work
- Examples (using scripts or tools) of isolation achieved on a Linux system

## What is a Container?

A container is a lightweight, portable and isolated environment that includes everything needed to run an application. It includes the application code, runtime, system tools, libraries and dependencies. Containers allow applications to run in a consistent environment across different systems, making them easy to deploy and manage.

For example, a container for a Node.js application would include a Linux distribution userland, Node.js runtime, application code, and any dependencies. The container can be run on any system, without needing to install Node.js or any other dependencies.

A container provides

- Virtualisation
- Isolation
- Resource Management

Virtualisation allows proper working of the application in a consistent environment, regardless of the underlying system. This is important, because if the application is run on different systems, it may not work properly due to differences in the underlying system.

Isolation ensures that the application runs in its own environment, without interfering with other applications. This is important, because if there is a vulnerability in one application, it will not affect other applications running on the same system, or the host system itself.

Resource management ensures that the application does not consume more resources than it is allowed. This is important, because if one application consumes too many resources, it can affect the performance of other applications running on the same system.

Containers are isolated from one another and bundle their own software, libraries and configuration files; they can communicate with each other through well-defined channels. Because all of the containers share the services of a single operating system kernel, they use fewer resources than VMs.

## How Containers Work

Like everything else in software, a container is just an abstraction over a set of features provided by the underlying system. In this case, the underlying system is the Linux kernel, which provides the necessary features to create and manage containers.

Containers are built using Linux kernel features such as cgroups and namespaces. These features provide isolation and resource management, allowing multiple containers to run on the same system without interfering with each other.

- **Cgroups**: Control groups allow you to set and monitor limits for a group of _processes_. Different subsystems (resource controllers) allow to control different resources, such as CPU, memory, disk I/O, etc.

- **Namespaces**: Namespaces wrap a global resource in an abstraction, such that to processes within the namespace, it appears that they have their own isolated instance of the resource.

  - PID namespace: Isolates the PID table, so that processes in different namespaces have different PID numbers.
  - Network namespace: Isolates network interfaces, routing tables, firewall rules, etc.
  - Mount namespace: Isolates the filesystem mount points, so that different namespaces can have different views of the filesystem.

Together, cgroups and namespaces provide the isolation and resource management needed to run containers.

### Control Groups

The Linux kernel provides a feature called _control groups_ (cgroups) that allows you to group processes together and apply resource limits to them. Cgroups are used to control and monitor the resource usage of a group of processes.

Docker creates control groups for each container, allowing you to set limits on CPU, memory, disk I/O, and other resources. This ensures that containers do not consume more resources than they are allowed.

Examples:

- Limit the amount of CPU time available to a container
- Limit the amount of memory available to a container
- Limit the amount of disk I/O available to a container

### Namespaces

Linux namespaces provide isolation for different resources, such as process IDs, network interfaces, and filesystem mount points. Namespaces allow you to create an isolated environment for a group of processes, so that they cannot interfere with processes outside the namespace.

A docker container is isolated, since it is an abstraction over a set of namespaces. Each container has its own PID namespace, network namespace, and mount namespace, among others. This ensures that containers are isolated from each other and from the host system.

- A PID namespace isolates the process ID table, so that processes within a namespace has its own set of PID numbers.
- A network namespace isolates network interfaces, routing tables, firewall rules, etc. This allows containers to have their own network stack, separate from the host system.
- A mount namespace isolates the filesystem mount points, so that containers have their own view of the filesystem.

## What is Docker?

Docker is an open-source platform that allows developers to _build_, _package_, and _run_ applications inside containers.

Docker is a higher level ecosystem which has tools to package and distribute containerised applications.

​Docker is an open-source platform that enables developers to build, deploy, and manage applications within containers—lightweight, portable units that package an application along with its dependencies and environment.

By utilizing containerization, Docker ensures that applications run consistently across various computing environments, enhancing portability and reducing conflicts between different system configurations. It leverages OS-level virtualization to deliver software in packages called containers, which are isolated from one another and bundle their own software, libraries, and configuration files. ​

## Docker Ecosystem

Two important tools in the Cloud Native ecosystem are `runc` and `containerd`.

- **containerd**: A container runtime that manages the complete container lifecycle - pulling images, managing storage, and running containers.

  - Handles image management (pulling, pushing, and storing images).
  - Manages container execution, including starting, stopping, and pausing containers.
  - Provides snapshot management (for storage backends like overlayfs).
  - Uses runc as the default runtime to create containers.

- **runc**: A lightweight container runtime that creates and runs containers based on the OCI (Open Container Initiative) specifications.
  - Implements the low-level container management tasks, such as creating namespaces, cgroups, and setting up the container environment.
  - Executes the container process in the configured environment.
