# vSphere HA Host Clusters

## Motivation

Data centers can have planned downtime for hardware maintenance, server migration, and firmware updates. 
An unplanned downtime (god forbid) caused from hardware or application failures.

We would like to minimize the impact of planned or unplanned downtime.

## Clusters

vSphere HA clusters enable a collection of ESXi hosts to work together so that, as a group,
they provide higher levels of availability for virtual machines than each ESXi host can provide
individually.

## Creating a HA cluster

> [!NOTE]
> - HA clusters require at least 2 hosts. Follow [ESXi installation](esxi_intro.md) and [Registering host into vCenter](vsphere_intro.md) to create your second ESXi host.
> - Before starting, delete all existed VMs in your data center.  

First, let's create an empty cluster.

1. In the vSphere Client, browse to your **data center**, and click **New Cluster**.
2. Complete the **Basics** page, as follows:
   - As this cluster will store workload related to the sample Netflix app, you can call it for example `john-netflix-service`.
   - Do not turn on vSphere HA, DRS or vSAN yet.
   - Since all ESXi hosts run the same ISO image (`8.0 GA - 20513097`), to can leave the **Manage all hosts in the cluster with a single image** enabled.
3. In the **Image** page, choose our ESXi host image version: `8.0 GA - 20513097`.
4. Finish the **New Cluster** wizard. 
5. Drag your hosts into your cluster (it's recommended to set the hosts to **maintenance mode**).
6. Browse to the cluster and enable vSphere HA.
   - Click the **Configure** tab.
   - Select **vSphere Availability** and click **Edit**.
   - Toggle the **vSphere HA** option to be enabled.
7. With **Enable Host Monitoring** enabled, hosts in the cluster can exchange network heartbeats and vSphere HA can take action when it detects failures. 
   Keep the recovery configurations at their default settings for now, we’ll experiment with them later. 
8. Under **Service**, choose **vSphere DRS**, click **Edit** and enable **vSphere DRS**.

Using vSphere HA with DRS combines automatic failover with load balancing.
This combination can result in a more balanced cluster after vSphere HA has moved virtual machines to different hosts.

8. Choose your cluster, and click on the **Monitor** tab. Under **Tasks and Events** you can monitor progress of setting your HA configurations.
   Upon successfully configurations, you shouldn't see any issues and alarms. 

Now all your cluster's hosts participate in an election to choose the primary host.

In addition, a small VM agent, named **vCLS** is provisioned first in the primary, and then in each secondary host of your cluster. 
Each vCLS agent is configured to communicate with other agents in the cluster.

### Troubleshooting 

The way to a healthy HA cluster might be long. The below section will help you to troubleshoot your cluster. 

> [!WARNING]
> ##### This host currently has no management network redundancy 
> A production grade cluster should have at least 2 VMKernel NIC, you can stay with 1 and peacefully live with the warning. 



> [!WARNING]
> #### No datastore configured for host
> You must configure a local datastore (at least for the vCLS VM agent)


> [!WARNING]
> If vCLS VMs fail to power on with an error message: 
> 
> ```text
> Feature 'MWAIT' was 0, but must be 1. Failed to start the virtual machine. Module FeatureCompatLate power on failed. 
> ```
> 
> The vCLC VMs are provisioned with **per VM EVC** feature enabled. Since we are working in a nested ESXi lab, you have to disable this feature. 
> 
> To apply the workaround, take the following steps:
> 
>  - Locate the vCLS VM (should be under `vCLS` dir) and open the **Configure** tab. Notice that **VMware EVC** option is **not** offered as an option.
>  - Identify the ESXi host where the vCLS VM resides (take a look on the **Related Objects** box in the **Summary** tab).
>  - Open the **ESXi Host Client** for this ESXi and login as root .
>  - Right click the vCLS VM within the host, and select **Upgrade VM Compatibility**, and click on **Upgrade**.
>  - Return to the vCLS VM in vCenter and click one the **Configure** tab for the VM. Notice that **VMware EVC** is now an option.
>  - Click **Edit** and choose **disabled**.  
>  - Another vCLS will power on the cluster note this.
> 
> For more information, [read here](https://knowledge.broadcom.com/external/article/326217/vcls-vms-fail-to-power-on-with-an-error.html). 

> [!WARNING]
> #### The number of vSphere HA heartbeat datastores for this host is 0, which is less than required: 2
> Don't worry, we will configure this later on. 


#### 🧐 Check yourself

Which one of your hosts is the primary? Which is secondary?

## Create a shared datastore

As mentioned, virtual machines in your cluster must be located on **shared, not local, storage**,
otherwise they cannot be failed over in the case of a host failure, and DRS cannot migrate them between hosts to balance the workload.

**Datastores**, to remind you, are logical containers that hide specifics of physical storage from virtual machines,
and provide a uniform model (in a VMFS format) for storing the virtual machine files. 

Currently, we use a traditional local hard disk based datastores for VMs located inside your ESXi host.

TODO figure vmhba0 SATA VMFS

A **storage area network (SAN)** is a specialized high-speed network that connects ESXi hosts to high-performance storage systems.
ESXi supports the common iSCSI protocol to connect to SAN systems.
The SCSI-based storage devices can be formatted as a regular VMFS datastores.

TODO figure iSCSI

In the ESXi context, the term **target** identifies a single **storage unit** that the host can access.

The terms **storage device** and **LUN** describe a logical volume that represents storage space on a target.

TODO explain each stuent has LUN id

![][vmware_storage_target_lun]

Logical Unit Number (LUN) within the SCSI target.

#### Allocate pysical storage and configure Configure the Software iSCSI Adapter

You can create a 200GB virtual iSCSI storage device by:

```bash
allocate-iscsi-storage-lun
```

You'll the as an output the target ID and LUN ID of your storage, to be used ... your cluster.

1. In the vSphere Client, navigate to your ESXi host.
2. Click the **Configure** tab.
3. Under **Storage**, click **Storage Adapters**, and click **Add Software Adapter**.
4. From the drop-down menu, select **Add iSCSI Adapter** and confirm that you want to add the adapter.

Now you need to set up target discovery addresses, so that your iSCSI storage adapter can
determine which storage resource on the network is available for access on your ESXi host.

1. Under **Storage**, click **Storage Adapters**, and select the iSCSI adapter (`vmhba#`) to configure.
2. In the adapter submenu, click **Dynamic Discovery** and click **Add**.
3. Enter the IP address of the storage system: `172.31.4.128`, and click **OK**.
4. Rescan the iSCSI adapter to dynamically discovery targets. 
5.  Under **Storage**, click **Storage Devices** and make sure you see a disk device with your LUN.

Repeat the above process for the other ESXi host. 

#### Configure a shared datastore for your cluster

1. Right-click on your cluster, under **Storage**, choose **New Datasource**.
2. In the **New Datastore** wizard, choose **VMFS**. Click **Next**.
3. Name it, for example `john-netflix-service-shared-ds`. Select one of your hosts, and choose your storage device (according to your LUN).
4. Continue with the default configurations to create the datastore. 
5. Right-click on the cluster, under **Storage**, choose **Rescan Storage** and your datastore should be mounted to your another host automatically. 

#### Configure heartbeat datastore 

All secondary hosts in the cluster sends a **heartbeat** to the primary host (every second), over the network.

When the primary host stops receiving these heartbeats from a secondary host, it checks 
whether the secondary host is exchanging heartbeats via a **shared heartbeat datastore(s)**, as an alternative to the fails network connection. 

If there are heartbeats with the datastore, the primary host assumes that the secondary host is a **network isolated**.
So, the primary host continues to monitor the host and its virtual machines through the heartbeat datastore.

Let's configure our shared datastores to be used for heartbeats.

1. Choose your cluster, and click the **Configure** tab.
2. In the configurations menu, under **Services**, choose **vSphere Availability**.
3. Click to edit the **vSphere HA** configurations. 
4. Move to the **Heartbeat Datastores** tab, choose your shared datastore as a heartbeat datastore. 


# Exercises 

### :pencil2: Migrate hosts to anotherclusters 

DRS must be enabled in the cluster for it to automatically migrate the VMs.
 If DRS is disabled, the VMs will need to be manually migrated, and HA will not be able to assist with VM placement.
vSphere HA will not be involved in the migration of VMs when manually taking the host out of the cluster. HA only takes action when a host failure occurs. However, it ensures that VMs are restarted elsewhere if the host fails during the operation.
If your VMs are on local datastores (not shared), you need to ensure that the data is accessible to other hosts before draining the host, either by using shared storage or replicating VM data across hosts.


maintenance mode 

https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vcenterhost.doc/GUID-32E79431-7DC9-48DB-A72C-CCA652A3B588.html

### :pencil2: check HA - shutdown host. 



### :pencil2: deletion of dswitch 

https://knowledge.broadcom.com/external/article/377194/migrate-vmkernel-adapter-from-distribute.html

- migrate VMkernel adaper in the standard switch 

make sure network uplink (physical adapters) redundency was not lost 

maintenance mode - try start VM ? insufficient failover level 


### :pencil2:

It protects against application failure by continuously monitoring a virtual machine and
resetting it in the event that a failure is detected.

It protects against datastore accessibility failures by restarting affected virtual machines on
other hosts which still have access to their datastores.

It protects virtual machines against network isolation by restarting them if their host becomes
isolated on the management or vSAN network. This protection is provided even if the
network has become partitioned.

### :pencil2: Migrate virtual machine 



### :pencil2: VM monitoring 

For VM Monitoring to work, VMware tools must be installed.

### :pencil2: vSphere Fault Tolerance (FT)

FT is a high availability (HA) feature that provides continuous availability for virtual machines (VMs) by creating a live shadow copy of a VM that runs on another ESXi host.

Unlike vSphere HA, which restarts VMs after a failure, FT ensures zero downtime by instantly switching to the secondary VM if the primary VM fails

Primary VM: Runs on one ESXi host and executes workloads.
2️⃣ Secondary VM: A mirrored, continuously synchronized copy of the primary VM runs on a different ESXi host.

Both VMs execute the same instructions simultaneously.
4️⃣ Failover: If the primary VM fails, the secondary VM takes over instantly, preventing downtime.

Synchronizes CPU and memory states between primary and secondary VMs.

How to Enable vSphere FT

1️⃣ Ensure Cluster is Configured for vSphere HA.
2️⃣ Verify Hosts Support FT (Check vSphere FT Compatibility).
3️⃣ Enable Fault Tolerance for the VM in vSphere Client:

    Right-click the VM → Select Fault Tolerance → Click Turn On Fault Tolerance.
    4️⃣ Configure a Dedicated FT Logging Network.
    5️⃣ Monitor FT Status in vCenter.

When to Use vSphere FT?

✔️ Mission-Critical Applications: Databases, financial systems, healthcare apps.


### :pencil2: maintenance mode exercise 

enter maintenance mode to drain machine for maintenance



Maintenance Mode in an ESXi host is a special state that allows you to perform updates, repairs, or other administrative tasks on the host without disrupting the overall operation of your environment.

When a host is placed into Maintenance Mode:

    VM Migration:
        All running virtual machines (VMs) are either migrated to other hosts in the cluster using vMotion or shut down if vMotion isn't available.
        No new VMs can be powered on the host.

    Host Management:
        Allows you to safely update ESXi, apply patches, upgrade hardware, or perform diagnostics.
        Prevents any unintended disruption to VMs during these activities.

    Cluster Awareness:
        Features like Distributed Resource Scheduler (DRS) ensure VMs are automatically redistributed across other hosts in the cluster before the host enters Maintenance Mode.




### :pencil2: enable VM and application monitoring 

### :pencil2: 

check data store failure by add block rule to firewall 

### :pencil2: 
see the addmision controller in action, remove host from cluster, try to turn on VM. 
disable the addimision controller, try to turn. 

### :pencil2: 

practice host isolation 

### :pencil2: practice host failure and host network isolation 

If a primary host cannot communicate directly with the agent on a secondary host, the secondary
host does not respond to ICMP pings. If the agent is not issuing heartbeats, it is viewed as
failed. The host's virtual machines are restarted on alternate hosts. If such a secondary host is
exchanging heartbeats with a datastore, the primary host assumes that the secondary host is in
a network partition or is network isolated. So, the primary host continues to monitor the host and
its virtual machines.


### :pencil2: 

Designate a separate network adapter for iSCSI.

### :pencil2: 

How vSphere Detects Isolation

    The host checks for heartbeat signals from other hosts.
    If no heartbeats are received, it tries to ping the isolation address (default: the default gateway).
    If both fail, the host is declared isolated.

### :pencil2: 

Enable VM Monitoring (VMware Tools Heartbeat)

    If the VM itself becomes unresponsive (not just the host), VM Monitoring will detect missing VMware Tools heartbeats and restart the VM.
    Go to vSphere HA Settings → VM Monitoring and enable it.

[vmware_storage_target_lun]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_storage_target_lun.png
