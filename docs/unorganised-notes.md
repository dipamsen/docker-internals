## Relevant commands/functions

- https://man7.org/linux/man-pages/man7/namespaces.7.html

Namespaces wrap a global resource in an abstraction, such that to processes within the namespace, it appears that they have their own isolated instance of the resource. Eg. PID namespace, where the global resource is the PID table.

PID namespace formation:

- use `clone()` system call with `CLONE_NEWPID` flag

```c
#include <sched.h>

clone(fn, stack, flags | CLONE_NEWPID, arg);
```

Unlike `fork()`, `clone()` can precisely control what pieces of execution context are shared between parent and child. Eg. whether they share virtual memory, file descriptors, signal handlers, etc.
Also, `clone()` can create a new process in a new namespace.

Working of `clone()`:

- Call `clone()` -> new process created
- Child process commences execution at `fn(arg)`
- When `fn()` returns, child process terminates. exit status is returned to parent process
- `stack` is the location of the stack for the child process, set up by the parent process (no way to specify stack size)

Other ways of creating PID namespace:

- use `unshare()` system call with `CLONE_NEWPID` flag

```c
#include <sched.h>

unshare(CLONE_NEWPID);
```

This creates a new PID namespace. The calling process is NOT moved into the new namespace.

User namespaces isolate security-related identifiers and
attributes- user IDs and group IDs, the root directory, keys, and
capabilities. A process's user and group
IDs can be different inside and outside a user namespace. In
particular, a process can have a normal unprivileged user ID
outside a user namespace while at the same time having a user ID
of 0 inside the namespace; in other words, the process has full
privileges for operations inside the user namespace, but is
unprivileged for operations outside the namespace.

- can be nested

## Cgroups - Control Groups

- https://man7.org/linux/man-pages/man7/cgroups.7.html

A cgroup is a group of processes, bound to a set of limits.

- A subsystem or resource controller modifies the behaviour of a process in a cgroup. There are many subsystems, eg. CPU, memory, blkio, etc, which allow to do - limit amount of CPU time and memory available to a cgroup, etc.

- cgroups for a controller are organised in a hierarchy, defined by the cgroup filesystem. At each level of the hierarchy, _attributes_ (eg. limits) can be defined. These have effect throughout subhierarchy, so limits at a higher level cannot be exceeded by a lower level.

- A process is in a cgroup for some controller. It may be in a different cgroup (in a different hierarchy) for another controller.

Create cgroup: `mkdir /sys/fs/cgroup/memory/mycgroup`

Move process to cgroup: `echo $$ > /sys/fs/cgroup/memory/mycgroup/cgroup.procs`

- writing 0 will move the writing process to the cgroup
- all threads of a process are moved to the cgroup

- remove cgroup: `rmdir /sys/fs/cgroup/memory/mycgroup` (it must have no child cgroups or processes)

v2: unified hierarchy

- processes may only reside in leaf cgroups or root cgroup

each cgroup has

- `cgroup.controllers`: controllers available for this cgroup
- `cgroup.subtree_control`: controllers enabled for this cgroup and its descendants
- write to `cgroup.subtree_control` as `echo "+memory" > x/y/cgroup/cgroup.subtree_control`
- when a controller is available, corresponding interface files are created in the cgroup directory (eg. `memory.max`)

## Network Namespaces

- https://man7.org/linux/man-pages/man8/ip-netns.8.html

```bash
ip link # list network interfaces
```

```bash
sudo unshare --net bash # create a new network namespace and run bash in it
```

instead, can use `ip netns` which manages network namespaces

```bash
ip netns add myns # create a new network namespace
ip netns exec myns ip link # list network interfaces in the namespace

ip link add veth0 type veth peer name veth1 # create a pair of virtual ethernet devices
ip link set veth0 netns myns # move one end of the pair to the namespace
ip netns exec myns ip link set veth0 up # bring the interface up
ip netns exec myns ip addr add 192.168.1.2/24 dev veth0 # assign an IP address to the interface
...
```

This enables network isolation between different network namespaces.

Docker uses network namespaces to provide network isolation between containers. Each container gets its own network namespace, with its own network interfaces, routing tables, firewall rules, etc.

Then, it sets up NAT (firewall) and network bridges, which forwards traffic on host ports to container ports. Thus, a service running in a container can be accessed from the host machine on a specific port.

## Union FileSystem

- https://man7.org/linux/man-pages/man8/mount.8.html

A docker image is a representation of a filesystem, comprising of the userland of a Linux distribution, application code, and dependencies.

Docker has the concept of Layers. Each layer represents a set of changes to the filesystem.

A union filesystem provides a unified view of multiple filesystem. Layers are implemented using ufs, which allows base layers to be shared between images. `Overlay` is a built-in ufs.

```bash
mount -t overlay overlay -o lowerdir=/lower,upperdir=/upper,workdir=/work /merged
```

Here `/merged` represents the union of `/lower` and `/upper` directories. User facing changes are done to `/merged`, which are stored in `/upper`.
