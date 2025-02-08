# vSphere cluster storage with vSAN


## Overview

So far, we've seen traditional storage virtualization models: 

- **Local storage** - internal hard disks connected to each of your ESXi host:

![][vmware_storage_local_dstastore]

- **Networked Storage (a.k.a. SAN)** - external storage systems that your ESXi host uses:

![][vmware_storage_iscsi_dstastore]


In addition to the above traditional models, VMware supports **virtualized
shared storage**, such as vSAN.

vSAN transforms local storage resources of your ESXi hosts into **single shared datastore**.
You can see it as a distributed layer of software, that aggregates and manages local disks across the cluster, creating
a single storage pool that can be used by every host in the cluster, even if the VM's data is not physically stored on that host.

vSAN runs natively as a part of the ESXi hypervisor and provides the capabilities required by HA cluster, such as vMotion for virtual machines.


## Core concepts

### Disk group 

On each ESXi host that contributes its local devices to a vSAN cluster, devices are organized into **disk groups**.
Each disk group must have one **flash (typically SSD) cache device** and one or multiple **capacity devices** (SSDs or HDDs).

Cache devices in vSAN temporarily store frequently accessed data to speed up read and write operations,
while capacity devices store all the persistent data.

![][vmware_vsan1]

### Failures to Tolerate (FTT) in vSAN cluster

First let's define **Objects** and **Components**. 

Unlike traditional VMFS datastores, vSAN does not use a file system, instead, it treats the stored elements as **object**.

Objects represent logical units of data (e.g. Virtual machine disk `vmdk` file) that are distributed across the cluster.
Each object (if bigger enough), is split into smaller **components** stored on different hosts for redundancy and availability.

![][vmware_vsan2]

In the above figure, the vmdk object is split into 3 components and replicated in 2 different hosts.

In addition to the object replicas, we can see another component (denoted by "W") which is the **Witness**. 

The witness does not store any data.
It only keeps metadata about the two replicas, and in case of data failure, or inconsistency between replicas.
If one of the hosts goes down, the witness component will determine which of the two replicas should be considered valid.

The above figure illustrates a vSAN storage policy of **RAID-1 Mirroring** with **Failures to tolerate (FFT)** set to 1.
In this policy, data is stored as two identical copies (replicas) across separate hosts, ensuring redundancy.
The policy allows the system to tolerate the failure of one host or one replica without data loss, as the second replica remains available for access.

In the above example, an object is considered **healthy** When at least one full RAID 1 mirror is available.



In addition to RAID-1 redundancy, vSAN offers RAID 5/6 policy.
Unlike RAID-1 (which simply mirrors data), RAID 5/6 uses technique called **erasure coding** to provide the **same level of data protection** as mirroring (RAID 1),
while using less storage capacity (RAID-5 offers up to 50% more storage efficiency than RAID-1, RAID-6 offers up to 33% more storage efficiency).

> [!NOTE] 
> If RAID-1 implies storage efficiency of 50% (For every 1TB of data, you need 2TB of total storage),
> RAID-5 provides `(N-1)/N` storage efficiency (where `N` is the number of disk groups).
> RAID-6 provides `(N-2)/N` storage efficiency.

### Data movement due to operational events

Operational events such full host evacuation may require vSAN to reconstruct object components to satisfy the policy. 

![][vmware_vsan4]

### Fault domains

Fault Domain in vSAN refers to a logical grouping of hosts within a vSAN cluster that are at risk of experiencing a common failure due to shared physical resources or infrastructure. 

vSAN ensures that replicas of data are placed across different fault domains.

When you place two hosts under the same fault domain in a vSAN cluster, it means that the hosts are considered vulnerable to a shared failure event.
For example, if both hosts are in the same physical rack, and that rack loses power, both hosts could go down simultaneously, which would impact the availability of the data stored on them.

![][vmware_vsan5]

If a host is not a member of a fault domain, vSAN interprets it as a stand-alone fault domain.


## Create vSAN Original Storage Architecture cluster

vSAN cluster can be provisioning in different architectures. 

We'll build the [Original Storage Architecture](https://techdocs.broadcom.com/us/en/vmware-cis/vsan/vsan/8-0/vsan-planning/building-a-virtual-san-cluster.html)

### Prerequisites 

1. You need a HA cluster with a minimum of **three ESXi hosts**, then vSAN is enabled on the cluster.
2. VMkernel Network must allow vSAN traffic in all hosts.
3. Your ESXi hosts must have one free SSD disk for cache, and one HDD or SSH for capacity. 
   The disk must be unformatted and unclaimed. If not, rescan the storage adapter to detect the available disks.
   
   To "buy and plug" cache and capacity disks to your host, you should run the `add-vsan-disks YOUR_ESXI_HOST_NAME` (change `YOUR_ESXI_HOST_NAME` according to esxi name) command.
   Then use the `virt-manager` to restart your ESXI nested VM to apply the new disks attachment. 
2. **HA functionality must be disabled during the creation and configuration of vSAN.**
1. Clean the iSCSI datastore built in the previous tutorial. You can also remove the storage adapter.

> [!NOTE]
> You can use the **Quickstart** wizard to quickly create and configure a vSAN cluster, but **don't** do it.


### Creating a vSAN cluster

1. Navigate to your host cluster.
2. Click the **Configure** tab.
3. Under **vSAN** choose **Services**. vSAN is turned off.
4. Select a configuration type to get started, choose **I need a local vSAN datastore**.
5. Choose **Standard vSAN cluster**., and click **Configure**.
6. Since your cluster is not ESA compatible, keep it disabled and click **Next**.
7. As this cluster is used for lab purposes, you can ignore the "host physical memory compliance check" warning, warns that our hosts isn't strong enough.
8. In the **Services** page, use the default configurations and click **Next**.
9. In the **Claim disks** page, define the SSD (flash) 100GB disk as the cache tier, and the 1000GB disk as the capacity tier, for all hosts.
10. Complete the wizard to create the cluster.

To verify success configuration, go to the **Configure** tab in your cluster, 
and under **vSAN** -> **Disk Management**, you should have 3 healthy shared disks.
In addition, you should see a shared datastore (typically called `vsanDatastore`), available to all your hosts. 

Don't forget to enable HA after vSAN cluster creation. 

> [!IMPORTANT]
> After creating the vSAN cluster, please make sure to carefully read the warnings and alarms in the cluster.
> It's important that you understand the warnings, if they can be avoided or there some error existed. 
> 
> To check the health status, go to **Monitor** tab in your cluster, and under vSAN, choose the **Skyline Health** page.

> [!NOTE]
> A vSAN datastore cannot be used for datastore heartbeating.
> Therefore, since no other shared storage is accessible, we won't use heartbeat datastores.


## Deploy a VM

To check the usability of your vSAN cluster, create a AlmaLinux VM using your vSAN datastore. 

# Exercises 


### :pencil2: Fail a host in a vSAN cluster

Let's simulate a host failure scenario in a vSAN cluster and understand the impact on data availability.

1. Select Enter Maintenance Mode or Shut Down to simulate the failure.
2. Make sure you choose Ensure Accessibility during maintenance mode to allow the VM data to remain accessible.


What happens to the virtual machine's data when the host is shut down or placed in maintenance mode?


vSAN will automatically rebuild the data to other available hosts if the data is replicated or mirrored.



Using Esxcli Commands with vSAN
Use Esxcli commands to obtain information about vSAN OSA or vSAN ESA and to troubleshoot
your vSAN environment.
The following commands are available:
Command Description
esxcli vsan network list Verify which VMkernel adapters are used for vSAN communication.
esxcli vsan storage list List storage disks claimed by vSAN.
esxcli vsan cluster get Get vSAN cluster information.
esxcli vsan health Get vSAN cluster health status.
esxcli vsan debug Get vSAN cluster debug information.

esxcli vsan health cluster list

- remove from vsan cluste: `esxcli vsan cluster leave` when the host is in maintenance mode in the cluster (dont take out)
- `esxcli vsan cluster get`
- `esxcli vsan storage list`
- clear partition t


[vmware_storage_local_dstastore]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_storage_local_dstastore.png
[vmware_storage_iscsi_dstastore]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_storage_iscsi_dstastore.png
[vmware_vsan1]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_vsan1.png
[vmware_vsan2]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_vsan2.png
[vmware_vsan4]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_vsan4.png
[vmware_vsan5]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_vsan5.gif

