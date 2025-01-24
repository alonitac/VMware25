# Single ESXi networking stack 

### Subnets

Networks organize machines into **subnets**.
All machines on a given subnet are connected by a physical or virtual device called **switch** and may exchange information directly.
Subnets are in turn linked to other subnets by **router** devices.

![][networking_subnets]

The above network consists of 3 subnets. Taking a closer look, we notice that computers under the same subnet have the same ip prefix.
For example, all computers under the leftmost subnet (and all computers that will join this subnet) have an IP address starting by `10.1.1.xxx`. 
Thus, they share the same IP prefix, more precisely, the same first **24 bits** in their IP address.

We will denote the IP boundaries of the leftmost subnet by:

```text
10.1.1.0/24
```

This method is known as **Classless Interdomain Routing (CIDR)**.

The CIDR `10.1.1.0/24` represents a network address in IPv4 format with the network prefix length of 24 bits. This means that the first 24 bits of the IP address, i.e., the first three octets, specify the **network portion**, and the remaining 8 bits, i.e., the fourth octet, represent the **host portion**. 
In this case, the network address is `10.1.1.0`, and there are 256 possible host addresses (2^8 = 256) within this network, ranging from `10.1.1.1` to `10.1.1.254` (the first and last IP addresses are reserved).

Another method to denote network subnet in **subnet mask**.
This format specifies the number of fixed octates of the IP as 255, and the free octates as 0. The equivalent subnet mask for `10.1.1.0/24` is `255.255.255.0`,  which means the first 3 octets are the network portion (fixed) and 4th octet is the hosts portion (change per machine in the subnet).

Use [this nice tool](https://cidr.xyz/) to familiarize yourself with CIDR notation.

### Default gateway

A **default gateway** is a network device, usually a **router**, that allows devices to communicate with networks outside their local subnet.
Its IP address is typically the first or last usable address in the subnet, such as `192.168.1.1` in a `192.168.1.0/24` network.

### Network interfaces

A **network interface** is a hardware or software component that allows a device to connect to a network and communicate with other devices over that network.
It can be physical or virtual.

### New IP address allocation and the DHCP 

In order to obtain a block of IP addresses for use within a subnet, a network administrator might first contact its ISP,
which would provide addresses from a larger block of addresses that had already been allocated to the ISP. 
In general, IP addresses are managed under the authority of the Internet Corporation for Assigned Names and Numbers ([ICANN](https://www.icann.org/)).

The Dynamic Host Configuration Protocol (DHCP) allows a host to obtain an IP address automatically within a subnet.

![][networking_dhcp1]

The below diagram examines the process by which a new client receiving an IP address from a DHCP server:

![][networking_dhcp2]

#### The Discover, Offer, Request, Acknowledge (DORA) process

DORA process refers to the four steps that are involved in DHCP client-server interaction. These four steps are:

- **Discover**: The client broadcasts a DHCP discover message on the local network, requesting an IP address lease from any available DHCP server.
- **Offer**: DHCP servers on the local network respond to the broadcast with a DHCP offer message, which includes an available IP address, lease duration, and other configuration parameters.
- **Request**: The client selects one of the offered IP addresses and sends a DHCP request message to the DHCP server, requesting the lease.
- **Acknowledge**: If the DHCP server approves the client's request, it sends a DHCP acknowledge message to the client, confirming the IP address lease and providing any other configuration parameters.

The DORA process allows a DHCP client to obtain an IP address and other configuration information from a DHCP server. It is an important part of network configuration, as it enables automatic IP address assignment and ensures consistency in network settings across multiple devices.


### IP Address for private subnets

There are three ranges of private IP addresses defined in [RFC 1918](https://www.rfc-editor.org/rfc/rfc1918):

- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

These addresses can be used for internal networks within an organization, but they are not routable on the public Internet.

Public IP addresses, on the other hand, are assigned by Internet authorities and are used to identify devices that are directly accessible from the Internet.

# Exercises 

### :pencil2: Network topology for single ESXi host

Build the following network stack for your environment:

The ESXi host uses two vSwitches to separate traffic types:

- vSwitch0 handles VM traffic (for both dev and prod) and is connected to the primary physical NIC (`vmnic0`).
  Production environment is VLAN ID `10`, development env is VLAN ID `20`.

![][esxi_network_ex1]

- vSwitch1 is dedicated to host management traffic, and is connected to a separate physical NIC for isolation (`vmnic1`).
  The VMkernel NIC must allow traffic to the following services: 
   - vMotion
   - Management
   - Fault tolerance logging

![][esxi_network_ex2]

After applying the configuration, you should be able to access the ESXi Client UI from the new `vmk1` address.

### :pencil2: Investigate the DHCP logs 

From the ESXi Client UI, go to **Monitor** -> **Logs** -> **DHCP client logs**

Search for logs that **explicitly** indicate the DORA process including Discover, Offer, Request, and Acknowledge messages, to verify the assignment of the new ESXi Client UI IP address.


[networking_subnets]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/networking_subnets.png
[networking_route]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/networking_route.png
[networking_dhcp1]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/networking_dhcp1.png
[networking_dhcp2]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/networking_dhcp2.png
[esxi_network_ex1]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/esxi_network_ex1.png
[esxi_network_ex2]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/esxi_network_ex2.png