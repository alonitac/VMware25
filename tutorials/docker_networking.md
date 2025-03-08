# Docker Networking

## Networking in Linux - recap 

![][networking_subnets]


## Network interfaces

Linux represents every networking device, either physical or virtual one, as a **Network Interface**.

In linux, the `ip addr` command can show network interfaces:

```console
myuser@hostname:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 0a:5c:36:18:fe:82 brd ff:ff:ff:ff:ff:ff
    inet 223.1.1.1/24 metric 100 brd 223.1.1.255 scope global dynamic eth0
       valid_lft 3559sec preferred_lft 3559sec
    inet6 fe80::85c:36ff:fe18:fe82/64 scope link 
       valid_lft forever preferred_lft forever
```


The above output shows 2 available network interfaces in the machine.

- `lo` (loopback) is being used to facilitate communication between processes running on the same host (internal communication).
- `eth0` is being used for connecting a computer to a wired Ethernet network.

How does the kernel know the correct network interface for which to route packets? Let's take a closer look at the below route table…

#### Route Table and Default gateway

To remind you, that in order to communicate with machines outside of your local **subnet**, your machine must know the identity of a nearby **router**.
The router used to route packets outside of your local subnet is commonly called as a **default gateway**.

In the above network figure, `10.1.1.4` is the default gateway IP of the leftmost subnet, while `10.1.2.31` is the default gateway IP of the rightmost subnet.

The Linux kernel maintains an internal table which defines which machines should be considered local, and what gateway should be used to help communicate with those machines which are not. This table is called the **routing table**.

The `route` command can be used to display the system's routing table (`ip route` is a more modern command).

```console
myuser@10.1.1.1:~$ route -n
Kernel IP routing table
Destination    Gateway        Genmask         Flags Metric Ref    Use Iface
0.0.0.0        10.1.1.4       0.0.0.0         UG    100    0        0 eth0
10.1.1.0       0.0.0.0        255.255.255.0   U     100    0        0 eth0
```

We will start with the second route. The second route specifies that traffic destined for the `10.1.1.0/24` network should remain in the subnet, there is no need to route the traffic to any nearby router, since sent traffic is destined for a machine in the same subnet. The `0.0. 0.0` appears in the Gateway column indicates that the gateway to reach the corresponding destination subnet is unspecified.

The first route specifies that traffic destined for the `0.0.0.0/0` be routed to `10.1.1.4`, which is the IP address of the default gateway in this subnet (take a look at the above diagram). Note that a destination of `0.0.0.0/0` means “all the internet”, since any IP address will match this CIDR.  

Is there any issue here? Isn't the IP range defined by the two destinations overlap?  For example, the ip `10.1.1.5` matches both the first and the second routes.

## Docker network sandbox and drivers

Docker implement a virtualized layer for container networking, enables communication and connectivity between Docker containers, as well as between containers and the external network.
It includes all the traditional stack we know - unique IP address for each container, virtual network interface that the container sees, default gateway and route table. 

Below is the virtualized model scheme. In Docker, this model is implemented by [`libnetworking`](https://github.com/moby/libnetwork) library: 

![][docker_sandbox]

A **sandbox** is an isolated network stack. It includes Ethernet interfaces, ports, routing tables, and DNS configurations.

**Network Interfaces** are virtual network interfaces (E.g. `veth` ).
Like normal network interfaces, they're responsible for making connections between the container and the rest of the world. 

Network interfaces are connected the sandbox to networks.

A **Network** is a group of network interfaces that are able to communicate with each-other directly. 
An implementation of a Network could be a Linux bridge, a VLAN, etc. 

This networking architecture is not exclusive to Docker. Docker is based on an open-source pluggable architecture called the [**Container Network Model** (CNM)](https://github.com/moby/libnetwork/blob/master/docs/design.md). 

The networks that containers are connecting to are pluggable, using **network drivers**.
This means that a given container can communicate to different kind of networks, depending on the driver. 
Here are a few common network drivers docker supports:

- [`bridge`](https://docs.docker.com/network/bridge/): This network driver connects containers running on the **same** host machine. If you don't specify a driver, this is the default network driver.

- [`host`](https://docs.docker.com/network/host/): This network driver connects the containers to the host machine network - there is no isolation between the
  container and the host machine, and use the host's networking directly.

- [`overlay`](https://docs.docker.com/network/overlay/): Overlay networks connect multiple container on **different machines**,
  as if they are running on the same machine and can talk locally. 

- [`none`](https://docs.docker.com/network/none/): This driver disables the networking functionality in a container.

## The Bridge network driver

The Bridge network driver is the default network driver used by Docker.
It creates an internal network bridge on the host machine and assigns a unique IP address to each container connected to that bridge.
Containers connected to the Bridge network driver can communicate with each other using these assigned IP addresses.
The driver also enables containers to communicate with the external network through port mapping or exposing specific ports.

The [default bridge network](https://docs.docker.com/network/network-tutorial-standalone/#use-the-default-bridge-network) official tutorial demonstrates how to use the default bridge network that Docker sets up for you automatically. 

The [user-defined bridge networks](https://docs.docker.com/network/network-tutorial-standalone/#use-user-defined-bridge-networks) official tutorial shows how to create and use your own custom bridge networks, to connect containers running on the same host machine.

Complete both **Use the default bridge network** and **Use user-defined bridge networks** tutorials. 

## The Host network driver 

The Host network driver is a network mode in Docker where a container shares the network stack of the host machine.
When a container is run with the Host network driver, it bypasses Docker's virtual networking infrastructure and directly uses the network interfaces of the host.
This allows the container to have unrestricted access to the host's network interfaces, including all network ports. However, it also means that the container's network stack is not isolated from the host, which can introduce security risks.

Complete Docker's short tutorial that demonstrates the use of the host network driver:      
https://docs.docker.com/network/network-tutorial-host/

## IP address and hostname

By default, the container is allocated with an IP address for every Docker network it attaches to.
A container receives an IP address out of the IP pool of the network it attaches to. 
The Docker daemon effectively acts as a DHCP server for each container.
Each network also has a **default subnet** mask and **gateway**.

As you've seen in the tutorials, when a container starts, it can only attach to a single network, using the `--network` flag.
You can connect a running container to multiple networks using the `docker network connect` command.

In the same way, a container's hostname defaults to be the container's ID in Docker. 
You can override the hostname using `--hostname`.

## DNS services

Containers that are connected to the default bridge network inherit the DNS settings of the host, as defined in the `/etc/resolv.conf` configuration file in the host machine (they receive a copy of this file). 
Containers that attach to a custom network use Docker's embedded DNS server. 
The embedded DNS server forwards external DNS lookups to the DNS servers configured on the host machine.

Custom hosts, defined in `/etc/hosts` on the host machine, aren't inherited by containers. 
To pass additional hosts into container, refer to [add entries to container hosts file](https://docs.docker.com/engine/reference/commandline/run/#add-host) in the `docker run` reference documentation.

# Exercises

### :pencil2: NetflixFrontend, NetflixMovieCatalog, Cassandra

Your goal is to run the following architecture (locally):

![][docker_frontend_catalog_db]

- The NetflixFrontend and NetflixMovieCatalog should be connected to a custom bridge network called `public-net-1` network.
- In addition, the NetflixMovieCatalog and a [Cassandra container](https://hub.docker.com/_/cassandra) should be connected to a custom bridge network called `private-net-1` network.
- The NetflixFrontend should talk with NetflixMovieCatalog using the `netflix-catalog:8080` hostname.
- The NetflixMovieCatalog app should talk to the Cassandra using the `cassandra-db:9042` hostname.
- Only port 3000 (of the Frontend container) should be published to the host machine. Other ports don't need to be exposed externally, as containers on the same bridge network can communicate directly without host publishing. 

### :pencil2: Inspecting container networking

Run the `busybox` image by:

```bash
docker run -it busybox /bin/sh
```

1. On which Docker network this container is running?
2. What is the name of the network interface that the container is connected to, as it is seen from the host machine?
3. What is the name of the network interface that the container is connected to as it is seen from within the container?
4. What is the IP address of the container?
5. Using the `route` command, what is the default gateway ip that the container use to access the internet?
6. Create a new bridge network, connect your running container to this network.
7. Provide an evidence that the container has been connected successfully to the created network.
8. From the host machine, try to `ping` the container using both its IP addresses. Were you able?


[docker_sandbox]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_sandbox.png
[docker_cache]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_cache.png
[docker_frontend_catalog_db]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_frontend_catalog_db.png

[networking_subnets]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/networking_subnets.png
