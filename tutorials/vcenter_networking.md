# vSphere Cluster Networking

## Overview

A **vSphere distributed switch** acts as a logical single switch across all associated hosts in the cluster to provide centralized provisioning, administration, and monitoring of virtual networks.

The networking configuration that you create on vCenter Server (the **management plane**) is automatically pushed down to all
host proxy switches (the **data plane**).


![][vmware_networking2]

In this way you can configure a group of virtual machines to
share the same networking configuration by associating the virtual machines to the same
distributed port group.


In addition to the abstractions used in Standard Switch (a switch resides in a single host), 
such as port groups and switch uplinks,
DSwitch introduces additional two abstractions: **uplink port group** and **distributed port groups**.


#### Uplink port group

In ESXi, uplink is a logical placeholder that is mapped to the physical NICs (pNICs).
Multiple uplinks can be used for failover and load balancing policies.

An **uplink port group** (or **dvuplink port group**) is a group of uplinks from each host, with the same ID:

![][vmware_networking1]

As can be shown below, for a distributed switch with 2 hosts connected,
uplink port group number 1 contains both `vmnic0` of hosts 1 and 2. 

We can say that uplink port group 1 represents the group of all `vmnic0` in each host. 


#### Distributed port group

Distributed port groups allow you to configure a group of virtual machines to
share the same networking configuration (e.g. NIC teaming, failover, load
balancing, VLAN, etc...) and the configurations will propagate to all hosts.

In this way you can apply consistent configurations for VMs in all hosts that are associated with the distributed port group.


## Create a distributed switch

1. In the vSphere Client, right-click a data center from the inventory tree.
2. Under **Distributed Switch**, select **New Distributed Switch**.
3. Enter a name for you DSwitch, e.g. `john-netflix-dswitch`.
4. On the **Select version** page, select a distributed switch version 8, which is compatible with our ESXi host version. Click **Next**.
5. On the **Configure settings** page, configure the distributed switch settings:
   - Select **None**, for **Network Offloads Compatibility**.
   - As each ESXi host has 2 physical NICs connected, select 2 as the **Number of uplinks**.
   - Keep the **Network I/O Control** enabled so we could prioritize the network resources per VM in the future. 
   - Select the **Create a default port group** check box to create a new distributed port group for your VMs with default settings.
6. On the **Ready to complete** page, review the settings you selected and click **Finish**.


> [!NOTE]
> #### What is Network Offloads Compatibility
> 
> For better networking performance, Sphere allows to offload some networking functionality from the ESXi host to a data processing units (DPUs), also
> known as SmartNIC. 
> Example for features that can be offloaded are health checks, port mirroring, etc. 

Entering the Inventory page and navigate to the **Networks** tab, 
you should see your DSwitch with two distributed port groups: 

- You default distributed port group for VMs. 
- An uplink port group (name contains **DVUplinks**). 

Let's create another distributed port group for VMKernel management adapters. 

1. On the vSphere Client Home page, click Networking and navigate to the distributed switch.
2. Right-click the distributed switch and select Distributed port group > New distributed port
group.
3. Name it uniquely, e.g. `john-dpg-management`. Network labels must be unique in data center.
4. On the **Configure settings** page, set the general properties for the new distributed port group.
   - For **Port binding** choose **Static binding** to keep the same port assignment even when the host or VM is powered off. 
   - For **Port allocation** choose **Elastic** to allocate as many ports as needed in the group (depending on the number of VMs in the group).
     As can be seen in **Number of ports**, the default size in 8, and when all the ports are assigned, a new set of eight ports is created.
   - Keep the remaining configurations as default.

## Migrate Network Adapters to a DSwitch

Our goal now is to migrate all the networking stack of your hosts to the DSwitch. 

Essentially, in order to no loosing connectivity with the host, 
we first need to connect one physical NIC with an active uplink, then 
migrate a VMkernel adapter, then the VMs adapters. 

But vSphere allow to migrate everything at the same time.


1. On the vSphere home page, click **Networking** and navigate to the distributed switch.
2. Right-click the distributed switch and select **Add and Manage Hosts**.
3. On the **Select task** page, select **Add hosts**, and click **Next**.
4. On the **Select hosts** page, choose **Select All** to migrate all your hosts, and click **Next**.
5. On the **Manage physical adapters** page, assign `vmnic0` to `Uplink 1`, and `vmnic1` to `Uplink 2`.
6. On the **Manage VMkernel adapters** page, under **Adapters on all hosts** tab, click **Assign Port Group** and assign your `vmk0` adapter to the created VMKernel management distributed port group. 
7. On the **Migrate VM networking** page, you can migrate virtual machines to your distributed switch.
8. On the **Ready to Complete** page, review the settings and click **Finish**.


vCenter will migrate all network adapters to the distributed switch.


# Exercises


### :pencil2: Networking policies 

handle 1 vmnic to handle only vmontion traffic 

vmk1 only to vmontion with vmnic of it's own 



What is Teaming and Failover Policy
NIC teaming lets you increase the network capacity of a virtual switch by including two or more
physical NICs in a team. To determine how the traffic is rerouted in case of adapter failure,
you include physical NICs in a failover order. To determine how the virtual switch distributes
the network traffic between the physical NICs in a team, you select load balancing algorithms
depending on the needs and capabilities of your environment.
NIC Teaming Policy
You can use NIC teaming to connect a virtual switch to multiple physical NICs on a host to
increase the network bandwidth of the switch and to provide redundancy. A NIC team can
distribute the traffic between its members and provide passive failover in case of adapter failure
or network outage. You set NIC teaming policies at virtual switch or port group level for a
vSphere Standard Switch and at a port group or port level for a vSphere Distributed Switch.

The Load Balancing policy determines how network traffic is distributed between the network
adapters in a NIC team. vSphere virtual switches load balance only the outgoing traffic. Incoming
traffic is controlled by the load balancing policy on the physical switch.

## network for vmotion 



## Network segmentation using VLAN 

How to Use VLANs to Isolate
Network Traffic

VLANs let you segment a network into multiple logical broadcast domains at Layer 2 of the
network protocol stack.

(VLAN Tagging Modes)

frontend and 2 catalog api, database 

### :pencil2: Network uplink redundancy lost after migration

Fix it...


### :pencil2: deletion of dswitch 

https://knowledge.broadcom.com/external/article/377194/migrate-vmkernel-adapter-from-distribute.html

- migrate VMkernel adaper in the standard switch 

make sure network uplink (physical adapters) redundency was not lost 

maintenance mode - try start VM ? insufficient failover level 

### 
exercise - remove virtual machine from DSwitch, Remove Host.



### :pencil2: 

host profile - same configuration for host
using host profile? 

### :pencil2: 

remove VM, remove host
migrating VMkernel interfaces and physical NICs to a vSphere Distributed
Switch

Removing Hosts from a vSphere Distributed Switch



### :pencil2: uplinks failover


### :pencil2: vmk dedicated to vSAN only


### :pencil2: uplink load balancing 

(Route Based on IP Hash)

Load balancing Specify the way an uplink is selected.
n Route based on originating virtual port: Select an uplink based on the
virtual port where the traffic entered the distributed switch.
n Route based on IP hash: Select an uplink based on a hash of the
source and destination IP addresses of each packet. For non-IP packets,
whatever is at those offsets is used to compute the hash.
n Route based on source MAC hash: Select an uplink based on a hash of
the source Ethernet.
n Route based on physical NIC load: Select an uplink based on the current
loads of physical NICs.
n Use explicit failover order: Always use the highest order uplink from the
list of Active adapters which passes failover detection criteria.

[vmware_networking1]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_networking1.png
[vmware_networking2]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_networking2.png



