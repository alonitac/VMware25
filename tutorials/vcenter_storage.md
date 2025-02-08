# vSphere cluster storage with vSAN


## Overview

So far, we've seen traditional storage virtualization models: 

- **Local storage** - internal hard disks connected to each of your ESXi host:

![][vmware_storage_local_dstastore]

- **Networked Storage** - external storage systems that your ESXi host uses:

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

first clean the iSCSI datastore!

prerequisited
- you must turn off HA when creating and configuring vSAN.  
- add 2 disks to your ESXi host (rescan adapter)


click on your cluster
configure
under vSAN choose services


vSAN is turned off.
Select a configuration type to get started.

choose - I need a local vSAN datastore

either: Standard vSAN cluster
Each host is considered to reside in its own fault domain. You can have mixed storage option of vSAN with other types like vSAN Directs and PMem.

or: Two node vSAN cluster
Two hosts reside at one site and a witness host at another site. Witness host contains only metadata and does not participate in storage operations. The witness host cannot be used to run VMs.

ignore "host pysical mempty compilence check" "non compliente host"


 Select the services to enable.
Space efficiency
None
Compression only
Deduplication and compression
Encryption
Data-At-Rest encryption
Wipe residual data
Key provider
Data-In-Transit encryption
Rekey interval
Predefined intervals
Disk format options
Allow reduced redundancy
RDMA support


then define the SSD (flash) 100GB the cache tier
the 1000GB the capacity tier

make sure the 2 hosts appear


under disk management you should have 2 healthy shared disks. and shared datastor vsanDatastore




what is vSAN direct? 

cache + capacity - vsan architecture 

https://knowledge.broadcom.com/external/article/314009/vsan-health-scsi-controller-is-vmware-ce.html
https://knowledge.broadcom.com/external/article/326880/vsan-cluster-configuration-consistency.html
https://knowledge.broadcom.com/external/article/326483/vsan-health-service-performance-service.html
https://community.broadcom.com/vmware-cloud-foundation/discussion/vsan-health-warning-discussion


quorum mechanism to avoid split-brain scenarios.

Instead of vSAN, use traditional shared storage (e.g., NFS, iSCSI, vSAN iSCSI Target) between the two hosts.

vSAN is implemented directly in the ESXi hypervisor.
like HA Agent


high-performance devices like SSDs or NVMe drives


esxcli vsan health cluster list


- quick start
  - HA disabeld
  - hhosts are in maintenance mode (cant enter host into mode because vm running and other host is in maintenance mode already)
  - click revalidate in configuration quickstart window

- claim unused disks 


Size: The general recommendation is that the cache disk should be at least 10-20% of the total capacity disk size.

sudo qemu-img create -f qcow2 /var/lib/libvirt/images/alonit-1-vsan-cache.qcow2 10G
sudo virsh domblklist alinit1

<disk type='file' device='disk'>
      <driver name='qemu' type='qcow2' discard='unmap'/>
      <source file='/var/lib/libvirt/images/alonit-1-vsan-capacity.qcow2'/>
      <target dev='sde' bus='sata'/>
      <address type='drive' controller='0' bus='0' target='0' unit='4'/>
    </disk>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2' discard='unmap'/>
      <source file='/var/lib/libvirt/images/alonit-1-vsan-cache.qcow2'/>
      <target dev='sdf' bus='sata' rotation_rate='1'/>
      <address type='drive' controller='0' bus='0' target='0' unit='5'/>
    </disk>

- reboot machines via vsphere

- manual
  - check vsan in both vmk0
  - 
what is vSAN section
RAID

TURNNNNNNNNNNN OFFFFFFFFFFF HAAAAAAAAAAAAAAAAAAAAAAAAA

- remove from vsan cluste: `esxcli vsan cluster leave` when the host is in maintenance mode in the cluster (dont take out)
- `esxcli vsan cluster get`
- `esxcli vsan storage list`
- clear partition t


figure 
Single Site vSAN Cluster


VMware vSAN is a distributed layer of software that runs natively as a part of the ESXi
hypervisor.
vSAN aggregates local or direct-attached capacity devices of a host cluster and creates a single
storage pool shared across all hosts in the vSAN cluster. While supporting VMware features that
require shared storage, such as HA, vMotion, and DRS, vSAN eliminates the need for external
shared storage and simplifies storage configuration and virtual machine provisioning activities.


VMware vSAN uses a software-defined approach that creates shared storage for virtual
machines.
It virtualizes the local physical storage resources of ESXi hosts and turns them into pools of
storage that can be divided and assigned to virtual machines and applications according to their
quality-of-service requirements. vSAN is implemented directly in the ESXi hypervisor.
You can configure vSAN to work as either a hybrid or all-flash cluster. In hybrid clusters, flash
devices are used for the cache layer and magnetic disks are used for the storage capacity layer.
In all-flash clusters, flash devices are used for both cache and capacity.

You can activate vSAN on existing host clusters, or when you create a new cluster. vSAN
aggregates all local capacity devices into a single datastore shared by all hosts in the vSAN
cluster. You can expand the datastore by adding capacity devices or hosts with capacity devices
to the cluster. vSAN works best when all ESXi hosts in the cluster share similar or identical
configurations across all cluster members, including similar or identical storage configurations.






image shared storage 




https://williamlam.com/2013/11/how-to-run-nested-esxi-on-top-of-vsan.html


https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vsan-planning.doc/GUID-A80526C8-A941-4F84-9D44-D4B8B3914A95.html

https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vsan-planning.doc/GUID-A80526C8-A941-4F84-9D44-D4B8B3914A95.html



vSAN pools local storage across hosts but requires at least 3 ESXi hosts.


vSAN aggregates local storage devices from multiple ESXi hosts into a single, shared storage pool. The storage is block-level and typically used to store virtual machine files (like VM disks, snapshots, and configuration files).
vSAN provides high availability and data redundancy across hosts in a cluster by replicating data across multiple hosts or using RAID-like configurations (e.g., RAID 1 or RAID 5).


https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.virtualsan.doc/GUID-4B738A10-4506-4D70-8339-28D8C8331A15.html

Yes, SAN uses a network to connect storage devices (like disk arrays or SAN storage systems) to servers or hosts. This allows these servers to access the storage as if it were directly attached, even though it's over a network.

AN can use different types of network protocols, such as:

    Fibre Channel (FC): A high-speed network used to connect servers and storage devices.
    iSCSI (Internet Small Computer Systems Interface): A protocol that enables block-level access over IP networks (Ethernet).

In this case, the network acts as a communication layer, but the storage itself is still managed as a block-level device.


Even though the data is transmitted over a network, a SAN presents storage to the host (servers) as block-level storage, which means that:

    The server can treat the storage just like any other locally attached hard drive (or block device), even though it may be located remotely.
    The storage is presented as a raw block device (like a hard disk or SSD), and the server can perform read and write operations directly on it using low-level block commands (e.g., using the SCSI protocol).
    The OS or hypervisor running on the host can format and partition the storage, and create filesystems just like it would with a local disk.

SAN uses the network to provide storage access to servers, but this access is presented at the block level (like a local disk).



To set up a vSAN cluster, you need to add at least 2 disks per host: one for cache and one for capacity. These disks should not be used for ESXi OS or VM storage.


A vSAN datastore cannot be used for datastore heartbeating. Therefore, if no other
shared storage is accessible to all hosts in the cluster, there can be no heartbeat datastores in
use. However, if you have storage that is accessible by an alternate network path independent of
the vSAN network, you can use it to set up a heartbeat datastore.

803-storage-guide

Local storage does not require a storage network to communicate with your host. You need a
cable connected to the storage unit

Figure 2-1. Local Storage


Local storage does not support sharing across multiple hosts. Only one host has access to a
datastore on a local storage device. As a result, although you can use local storage to create
VMs, you cannot use VMware features that require shared storage, such as HA and vMotion.

However, if you use a cluster of hosts that have just local storage devices, you can implement
vSAN. vSAN transforms local storage resources into software-defined shared storage. With
vSAN, you can use features that require shared storage.

virtualized
shared storage, such as vSAN.



Local and Networked Storage
In traditional storage environments, the ESXi storage management process starts with
storage space that your storage administrator preallocates on different storage systems.
ESXi supports local storage and networked storage.
See What Types of Physical Storage Does ESXi Support.
Storage Area Networks
A storage area network (SAN) is a specialized high-speed network that connects computer
systems, or ESXi hosts, to high-performance storage systems. ESXi can use Fibre Channel or
iSCSI protocols to connect to storage systems.

Virtual Disk Thin Provisioning with vSphere Storage  (figure page 365)

Datastore Support
I/O filters can support all datastore types including the following:
n VMFS
n NFS 3
n NFS 4.1
n vVol
n vSAN

vCenter Server and ESXi support VMFS, NFS, vSAN, and Virtual Volumes datastores.

In the vSphere environment, datastores are logical containers, analogous to file systems, that
hide specifics of physical storage and provide a uniform model for storing virtual machine files.
You can also use datastores for storing ISO images, virtual machine templates, and floppy
images. vCenter Server and ESXi support VMFS, NFS, vSAN, and Virtual Volumes datastores.


To store virtual disks, ESXi uses datastores. The datastores are logical containers that hide
specifics of physical storage from virtual machines and provide a uniform model for storing the
virtual machine files. The datastores that you deploy on block storage devices use the native
vSphere Virtual Machine File System (VMFS) format. It is a special high-performance file system
format that is optimized for storing virtual machines.

Datastore Extents. on multiple devices


Monitor Physical Devices in vSAN Cluster
You can monitor hosts, cache devices, and capacity devices used in the vSAN cluster.
Procedure
1 Navigate to the vSAN cluster.
2 Click the Configure tab.
3 Click Disk Management to review all hosts, cache devices, and capacity devices in the cluster.
The physical location is based on the hardware location of cache and capacity devices on
vSAN hosts. You can see the virtual objects on any selected host, disk group, or disk and
view the impact of the selected entity to the virtual objects in the cluster.




Using Esxcli Commands with vSAN
Use Esxcli commands to obtain information about vSAN OSA or vSAN ESA and to troubleshoot
your vSAN environment.
The following commands are available:
Command Description
esxcli vsan network list Verify which VMkernel adapters are used for vSAN communication.
esxcli vsan storage list List storage disks claimed by vSAN.
esxcli vsan storagepool list List storage pool claimed by vSAN ESA. This command is applicable only for vSAN
ESA cluster.
esxcli vsan cluster get Get vSAN cluster information.
esxcli vsan health Get vSAN cluster health status.
esxcli vsan debug Get vSAN cluster debug information.





# Exercises 

### :pencil2: 

- who is the master? 


        A cluster should have only one host with the Master role. More than a single host with the Master role indicates a problem

        The host with the Master role receives all CMMDS updates from all hosts in the cluster


- who is backup role?


        The host with the Backup role assumes the Master role if the current Master fails

        Normally, only one host has the Backup role


- who is agent? 


        Hosts with the Agent role are members of the cluster

        Hosts with the Agent role can assume the Backup role or the Master role as circumstances change

        In clusters of four or more hosts, more than one host has the Agent role



### :pencil2: understand error and warning in skyline health 



### :pencil2: 

Designate a separate network adapter for iSCSI.

### :pencil2: 

deploy with witness

### :pencil2: data migration precheck 


### :pencil2: 

fix Disks usage on storage controller

[vmware_storage_local_dstastore]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_storage_local_dstastore.png
[vmware_storage_iscsi_dstastore]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_storage_iscsi_dstastore.png
[vmware_vsan1]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_vsan1.png
[vmware_vsan2]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_vsan2.png
[vmware_vsan4]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_vsan4.png
[vmware_vsan5]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_vsan5.gif

