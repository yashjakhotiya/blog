---
title: "Containers, Container Runtimes, and What Kubernetes 'Docker' Deprecation Really Means"
description: "Kubelet to Kernel Space, and Everything in Between"
layout: post
toc: true
comments: true
hide: false
search_exclude: false
image: /images/<tbd, creately.com>
categories: [containers, kubernetes]
author: "<a href='https://www.linkedin.com/in/yash-jakhotiya/'>Yash Jakhotiya</a>"
---

# Containers? Aren't they ... like some sort of virtual machines?

Docker popularized the notion of using containers - isolated environments leveraging OS-level virtualization where each running process sees the environment as one whole computer. This seems awfully similar to virtual machines, except that it isn't. Containers differ from virtual machines in that each container does not host the entire operating system the way virtual machines do. 

![](https://yashjakhotiya.github.io/blog/images/2020-12-20-container-runtimes/containers_vs_vms.png "Virtual Machines Vs Containers") 

Container images, which become running containers when instantiated, store the application code and any required dependencies mentioned in the image [Dockerfile](https://docs.docker.com/engine/reference/builder/). When you don't need to worry about dependencies, shipping applications from a developer's laptop to production servers or public cloud environments becomes easier. You _could_ package your application as a custom built VM image relying on a fully functional traditional OS packaged with it, but container images are much more lightweight and can be easily maintained.

# But, every dockerfile I see ultimately stems from an OS image. Doesn't that mean container images have their own OS installed?

This is an excellent question. Thanks for asking! To answer this, let us get some background context.

In most modern OS, an application process runs in what is known as a 'user mode'. The idea here is to restrict the memory area accessible to an application process and prevent it from accessing and potentially corrupting memory areas associated with kernel and other application processes. This is implemented using [Virtual memory](https://en.wikipedia.org/wiki/Virtual_memory) and [Protection Rings](https://en.wikipedia.org/wiki/Protection_ring), and assisted by hardware in the form of [Protected mode](en.wikipedia.org/wiki/Protected_mode) and [Memory Management Units](https://en.wikipedia.org/wiki/Memory_management_unit).

Providing fault tolerance and computer security with this form of memory protection effectively results in a typical OS being divided into two bifurcations - a user space and a kernel space. An application running in user space can access resources which it does not have direct access to (like I/O devices or files lying on a disk) with special requests to the kernel called [system calls](https://man7.org/linux/man-pages/man2/syscalls.2.html).

![](https://yashjakhotiya.github.io/blog/images/2020-12-20-container-runtimes/user_space_kernel_space.png "A process in user space makes a system call")

System calls serve as APIs for all 'userland' software to interact with the kernel. In the Linux world, all distros run (a bit simplification here) the same kernel. This makes it possible for the userland software coming from Ubuntu to talk to a CentOS kernel.

What you see inside a Dockerfile, which gets installed in the built image, is not a full-fledged OS. It is the trimmed-down version of the userland software of the OS, bare enough to talk to the host's kernel. It is not uncommon to see containers with Ubuntu, CentOS, and Debian base run parallely on a RHEL7 host.

# Ok. I am curious. How is this implemented?

To be honest, container implementation recipe is really not that difficult if you understand its three main ingredients - cgroups, namespaces and chroot. Let us focus on each of them below.

1. cgroups, or Control Groups is a Linux kernel feature. With cgroups you can allocate, monitor, and limit resources - like CPU time, memory, or network bandwidth - to a process or a collection of processes. Linux command [cgcreate](https://linux.die.net/man/1/cgcreate) helps you create a control group, [cgset](https://linux.die.net/man/1/cgset) sets resource limits for the control group, and with [cgexec](https://linux.die.net/man/1/cgexec) you can run a command in the control group.

2. A namespace is another Linux kernel feature, with which you can isolate a global _resource_. This creates an illusion of a separate instance of the resource to the processes running in the namespace, and any changes made are not visible outside! The resources you can abstract this way include process IDs, hostnames, user IDs, etc. [unshare](https://man7.org/linux/man-pages/man1/unshare.1.html) \[options\] \[_program_ \[arguments\]\] is a Linux utility you can use to create namespaces (supplied in `options`) and run `program` in it. For example you can create a UTS (Unix Time Sharing) namespace, which controls host and domain names, using the -u option as illustrated below. 

```
> hostname                              # show current hostname 
personal-ubuntu                                                                                                     > unshare -u /bin/sh                    # run a shell instance with UTS namespace unshared from parent
> hostname a-different-hostname         # change hostname to a-different-hostname
> hostname                              # verify that the hostname has been changed
a-different-hostname
> exit                                  # exit from the shell process, effectively destroying the namespace
> hostname                              # voila!
personal-ubuntu                         # changing the hostname inside the namespace has no effect outside!
```

3. [chroot](https://linux.die.net/man/1/chroot) is a Linux utility that can change the apparent root directory for a process and its children. Running `chroot NEWROOT command` will run `command` with `NEWROOT` as its apparent root directory. This modified environment is also called a 'chroot jail', because `command` can not name and hence can not normally access files outside `NEWROOT`.

Now that you have understood these three main concepts, let us say we created a container image from the following dockerfile, exported it into a tar file which we want to run.
```
FROM ubuntu:18.04
COPY script.py /app
CMD python /app/script.py
```
This container image contains the ubuntu:18.04 userland file structure heirarchy, /app/script.py and some environment configuration. Ignoring the config part for now, your minimal implementation can run this image in just 4 steps.

1. Export and extract contents of the image in `new_root_dir`
```
> mkdir new_root_dir
> docker export docker_image | tar -xf - -C new_root_dir
```
2. Create a control group and set memory and CPU use limits
```
> control_group=$(uuidgen)
> cgcreate -g cpu,memory:$control_group
> cgset -r memory.limit_in_bytes=50000000 $control_group
> cgset -r cpu.shares=256 $control_group
```
3. Executing inside the control group, call unshare to separate namespaces and execute `script.py` inside the `new_root_dir` jail
```
> cgexec -g cpu,memory:$control_group unshare -uinpUrf --mount-proc sh -c "chroot new_root_dir /app/script.py"
```

4. Cleanup. Delete the cgroup and `new_root_dir`. Unless bound to a file, namespaces cease to exist once all running processes in the namespace have exited.
```
> cgdelete -r -g cpu,memory:$control_group
> rm -r new_root_dir
```

Lo and behold! You have just created a minimal container runtime!

# Whoa! Wait, Container R... what?

Container Runtime - the code and tooling responsible for running containers. What you created above is the heart of what every container runtime does. Although, it catches the essence of container runtimes, it's still minimal. Docker images also have something known as a `config.json`. This file has, among other things, environment variables to be set for the running process inside the container, and the uid and gid of the user the process must run as.

The code to run containers used to be deep inside a monolith called `Docker`. But, it need not be. As long as vendors agreed upon a common specification for images and a common specification for runtimes, anybody could create runtimes customized to their needs. That's exactly what they did. Docker, CoreOS, Google and other industry leaders in the container space came together and launched [Open Container Initiative](https://opencontainers.org) in June 2015. OCI is responsible for defining [image-spec](https://github.com/opencontainers/image-spec) and [runtime-spec](https://github.com/opencontainers/runtime-spec), which every OCI-compliant image builder and container runtime has to abide by.

OCI even develops and maintains a reference implementation of the runtime-spec called [runc](https://github.com/opencontainers/runc). runc broke off from Docker, as part of the Open Container Initiative. Although, runc is self-sufficient to run containers, it is a low-level runtime. The only developers that work with runc are developers of high-level runtimes.

# Come on! These container runtimes have 'levels' now?

Yes, they very much do. If you have ever used docker, you might know that running containers from images isn't all that you do. You might want to pull images from registeries before you actually run them. A higher level runtime does that for you.

![](https://yashjakhotiya.github.io/blog/images/2020-12-20-container-runtimes/container_runtimes.png "A higher level runtime interacting with a lower level runtime")

Higher level runtimes are also responsible for unpacking the container image into an [OCI runtime bundle](https://github.com/opencontainers/runtime-spec/blob/master/bundle.md) before spawning a runc process to run it. In addition to managing the lifecycle of a container, higher level runtimes are also sometimes responsible for low level storage and network namespace management. This is usually in place to facilitate interaction between individual container processes. 

Humans aren't the only entities that interact with higher level runtimes. [Container orchestration](https://www.redhat.com/en/topics/containers/what-is-container-orchestration) services (just a fancy term for management and configuration of containers across large dynamic systems), like [Kubernetes](https://kubernetes.io), need to interact with high-level runtimes. For most industry use-cases, it's less humans and more such services that need to talk to higher level runtimes.

# Did you mention Kubernetes? You had my curiosity. Now you have my attention.

What interacts with high-level container runtimes are not client-facing modules of a running Kubernetes instance, but a node-agent called [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) which runs on all nodes in a Kubernetes cluster. Kubelet is responsible to ensure all containers mentioned in a pod's specification are running and healthy. It registers nodes, sends pod status and events, and reports resource utilization higher up the command chain.

With the introduction of OCI, many container runtimes came up that could support running OCI-compliant container images, and so arised the need for Kubernetes to support these runtimes. To avoid deep integration of such runtimes into kubelet source code, and the subsequent maintenance that would follow, Kubernetes introduced the [Container Runtime Interface](https://github.com/kubernetes/cri-api) - an interface definition which enables kubelet to use a wide variety of runtimes. It is the responsibility of a container runtime to implement this interface as an internal package or as a shim.

[containerd](https://github.com/containerd), a prominent high-level container runtime, which broke off from Docker similar to runc, recently graduated its [cri-shim](https://github.com/containerd/cri) codebase to its main [containerd/containerd](https://github.com/containerd/containerd) repository, marking CRI-implementation to be an important part of the container runtime. [cri-o](cri-o.io) is another implementation of CRI, focused and optimized only for Kubernetes, and, unlike containerd, can not service docker daemons for container orchestration.

![](https://yashjakhotiya.github.io/blog/images/2020-12-20-container-runtimes/crio-to-kernel.png "CRI-O to Kernel")

Now that we have established CRI, let us talk about what Kubernetes recent 'Docker Deprecation' really means.

# Finally!

Kuberenetes recently announced that it will be [deprecating Docker](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#deprecation). It really isn't as dramatic as it sounds. What Kuberenetes will not support is Docker as a **runtime**, and nothing else changes. Images built with dockerfiles are OCI-compliant and hence can be very well used with Kubernetes. Both containerd and cri-o know how to pull them, and runc knows how to run them.

Docker, being built for human interaction, isn't really friendly for Kubernetes as just a runtime. To interact with it, Kubernetes has to develop a module called "dockershim", which implements CRI support for Docker. This makes Docker callable by kubelet as a runtime. Kubernetes is no longer willing to maintain this, especially when containerd (which Docker internally uses) has a CRI plugin. If you are developer, you do not really need to worry about what runtimes kubelet can interact with. Docker built images are perfectly fine for Kubernetes to consume!

![](https://yashjakhotiya.github.io/blog/images/2020-12-20-container-runtimes/dockershim-containerd.png "Dockershim deprecation")

# End Notes

I hope you liked reading this blog post as much as I loved writing it. I'll soon update a _large_ list of references which you can use for further reading. In the meanwhile, please feel free to follow me on [Twitter](https://twitter.com/yash_jakhotiya) and subscribe to the Blog's [RSS Feed](https://yashjakhotiya.github.io/blog/feed.xml) for further updates. For any feedback or suggestions for blog post, please drop me a DM on Twitter, or [email](mailto:mailsforyashj@gmail.com) me. Thanks for reading!