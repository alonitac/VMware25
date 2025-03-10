# Docker Containers Intro

Containers offer a similar level of isolation for installing and configuring binaries/libraries as virtual machines, but they differ in their approach.
Instead of virtualizing at the **OS layer**, containers virtualize the **app layer** by utilize native Linux features like **cgroups** and **namespaces** to provide isolation while sharing the same kernel. 
This lightweight approach allows for faster startup times, improved resource efficiency, and easier scalability compared to traditional virtual machines.

![][docker_containers-vs-vms]


Containers are an abstraction at the app layer that packages code and dependencies together.
Multiple containers can run on the same machine and **share the OS kernel with other containers**, each running as isolated processes in user space. 
Containers take up less space than VMs (container images are typically tens of MBs in size), can handle more applications and require fewer VMs and Operating systems.

## Containers terminology

A **Container Image** (or shortly, an **Image**) is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.
You can transfer images from one machine to the other. Every machine is able to run this image without the need to install the application dependencies, define environment variables and networking settings.

A **Container** is a single running instance of an image. You can create, start, stop, move, or delete a container, and they can be run easily and reliably from one computing environment to another.
The computer that runs the container is frequently referred to as a **Host Machine**, because it "hosts" containers.

By default, a container is relatively well isolated from other containers and its host machine. You can control how isolated a container's network, storage, or other underlying subsystems are from other containers or from the host machine.

A container, as we mentioned, is defined by its image as well as any configuration options you provide to it when you create or start it.
When a container is removed, any changes to its state that are not stored in persistent storage disappear.

## Containers under the hoods

Under the hoods, containers are merely a **linux processes**. 

But they are unique processes that "live" in an isolated environment. By this means, the process by default cannot "see" or interact any other process in the host machine. It "believes" that he is the only process in the system. 
Containers are a technology that leverages the Linux kernel's features to provide lightweight and isolated environments for running applications. 

Linux containers utilize several key components:

- **Namespaces**: Linux namespaces provide process-level isolation by creating separate instances of various resources, such as the process ID namespace, network namespace, mount namespace, and more. Each container has its own isolated namespace, allowing processes within the container to have their own view of system resources.
- **Control Groups (cgroups)**: Control groups, or cgroups, enable resource management and allocation by imposing limits, prioritization, and accounting on system resources such as CPU, memory, disk I/O, and network bandwidth. Cgroups ensure that containers do not exceed their allocated resources and provide fine-grained control over resource utilization.

## Docker architecture

**Docker** is an open platform for developing, building and shipping images, and running containers.

Docker uses a client-server architecture. The Docker client talks to the Docker daemon, which does the work of building, running, and distributing your Docker containers. The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon. The Docker client and daemon communicate using a REST API, over UNIX sockets or a network interface.

![][docker_arc]

- The Docker daemon (`dockerd`) listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes.
- The Docker client (`docker`) is the primary way that many Docker users interact with Docker. When you use commands such as docker run, the client sends these commands to dockerd, which carries them out.
- A Docker registry stores Docker images. [Docker Hub](https://hub.docker.com/) is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default. You will even run your own private registry later on in the course.

### Docker installation

To install Docker, please **carefully** follow the [installation instructions](https://docs.docker.com/engine/install/ubuntu/) if you haven't done it before.

> [!TIP]
> You can add your user to the `docker` group, so you could use the `docker` command without `sudo`:
> 
> ```bash 
> sudo usermod -aG docker $USER
> ```

Upon up and running `docker` installation, the `docker info` command output may look like: 

```text
Client: Docker Engine - Community
 Version:           20.10.22
 API version:       1.41
 Go version:        go1.18.9
 Git commit:        3a2c30b
 Built:             Thu Dec 15 22:28:02 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.22
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.18.9
  Git commit:       42c8b31
  Built:            Thu Dec 15 22:25:51 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.14
  GitCommit:        9ba4b250366a5ddde94bb7c9d1def331423aa323
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## Hello world example 

The Docker "Hello World" container is a simple and lightweight container that is often used to verify if Docker is properly installed and functioning on a system. It is based on the official Docker image called [hello-world](https://hub.docker.com/_/hello-world).

```bash
docker run hello-world
```

The hello-world image is an example of minimal containerization with Docker.
It has a single `hello.c` file responsible for printing out the message you're seeing on your terminal.


> [!NOTE] 
> #### Docker under the hood
> 
> Note that docker is running as a service on your system, hence can be controlled by `systemctl`:
> 
> ```bash
> $ sudo systemctl status docker
> ● docker.service - Docker Application Container Engine
>      Loaded: loaded (/lib/systemd/system/docker.service; disabled; vendor preset: enabled)
>      Active: active (running) since Sun 2023-05-07 09:56:45 IDT; 5min ago
> TriggeredBy: ● docker.socket
>        Docs: https://docs.docker.com
>    Main PID: 261600 (dockerd)
>       Tasks: 123
>      Memory: 209.5M
>      CGroup: /system.slice/docker.service
>              └─261600 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
> 
> May 07 09:56:39 hostname dockerd[261600]: time="2023-05-07T09:56:39.509330916+03:00" level=warning msg="Your kernel does not support CPU realtime scheduler"
> May 07 09:56:39 hostname dockerd[261600]: time="2023-05-07T09:56:39.509350949+03:00" level=warning msg="Your kernel does not support cgroup blkio weight"
> May 07 09:56:39 hostname dockerd[261600]: time="2023-05-07T09:56:39.509365744+03:00" level=warning msg="Your kernel does not support cgroup blkio weight_device"
> May 07 09:56:39 hostname dockerd[261600]: time="2023-05-07T09:56:39.533460844+03:00" level=info msg="Loading containers: start."
> May 07 09:56:42 hostname dockerd[261600]: time="2023-05-07T09:56:42.942065068+03:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
> May 07 09:56:43 hostname dockerd[261600]: time="2023-05-07T09:56:43.248511892+03:00" level=info msg="Loading containers: done."
> May 07 09:56:44 hostname dockerd[261600]: time="2023-05-07T09:56:44.943477277+03:00" level=info msg="Docker daemon" commit=42c8b31 graphdriver(s)=overlay2 version=20.10.22
> May 07 09:56:44 hostname dockerd[261600]: time="2023-05-07T09:56:44.972157071+03:00" level=info msg="Daemon has completed initialization"
> May 07 09:56:45 hostname dockerd[261600]: time="2023-05-07T09:56:45.533037228+03:00" level=info msg="API listen on /var/run/docker.sock"
> May 07 09:56:45 hostname systemd[1]: Started Docker Application Container Engine.
> ``` 
> 
> From docker's service status output we can learn a few important properties of the docker client and daemon. 
> 
> When the Docker client (`docker`) and daemon (`dockerd`) are on the same machine (usually the case), 
> they communicate using a UNIX socket located in `/var/run/docker.sock`, typically via RESTful API endpoints.
> 
> When the client and daemon are not on the same machine, they communicate over the internet via HTTPS protocol.
> 
> What else can we learn about the docker daemon? that it does not run containers itself! Docker relies on the `containerd` service to manage containers lifecycle.
> Containerd is an open-source container runtime that provides a **high-level interface** for managing container lifecycle and execution. It is serving as the underlying runtime for various container platforms, including Docker.
> Containerd, in turn, uses `runc` as the default OCI-compliant runtime for actually running containers. Containerd utilizes runc to execute the container processes, manage resource isolation, and handle **low-level interface** container operations according to the OCI specification.
> 
> To summarize, containers are not exclusive to Docker, they are a broader technology and concept that existed before Docker's popularity. 
> Docker popularized and simplified the adoption of containers by providing a user-friendly interface and tooling, but there are alternative container runtimes and platforms available, such as Podman, that leverage containers for application deployment and management.
> 
> Under the hood, `runc` does the dirty job of running containers [^1]: 
> 
> ![][docker_under-the-hood]
> 
> In a moment, you'll run your first container, this is the execution order of the different components that responsible for the container execution: 
> 
> ![][docker_cont-run]


[docker_containers-vs-vms]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_containers-vs-vms.png
[docker_arc]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_arc.png
[docker_under-the-hood]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_under-the-hood.png
[docker_cont-run]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_cont-run.png

[^1]: Images by https://mkdev.me/posts/the-tool-that-really-runs-your-containers-deep-dive-into-runc-and-oci-specifications">Image source: https://mkdev.me/posts/the-tool-that-really-runs-your-containers-deep-dive-into-runc-and-oci-specifications