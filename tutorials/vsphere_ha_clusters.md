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

![][vmware_storage_shared_dstastore]

**Datastores**, to remind you, are logical containers that hide specifics of physical storage from virtual machines,
and provide a uniform model (in a VMFS format) for storing the virtual machine files. 

Currently, we use a traditional local hard disk based datastores for VMs located inside your ESXi host.

![][vmware_storage_local_dstastore]

A **storage area network (SAN)** is a specialized high-speed network that connects ESXi hosts to high-performance storage systems.
ESXi supports the common iSCSI protocol to connect to SAN systems.
The SCSI-based storage devices can be formatted as a regular VMFS datastores.

![][vmware_storage_iscsi_dstastore]

In the ESXi context, the term **target** identifies a single **storage unit** that the host can access.

The terms **storage device** and **LUN** describe a logical volume that represents storage space on a target.

![][vmware_storage_target_lun]

To practice shared datastores, each one of you create a Logical Unit Number (LUN) within the same SCSI target.

#### Allocate physical storage and Configure the Software iSCSI Adapter in your hosts

You can allocate a 200GB virtual iSCSI storage device by executing the following command from a terminal in our lab:

```console
$ allocate-iscsi-storage-lun
Creating disk at /var/lib/libvirt/images/ubuntu-shared-disk.raw...
Formatting '/var/lib/libvirt/images/ubuntu-shared-disk.raw', fmt=raw size=214748364800
Creating iSCSI logical unit (TID: 1, LUN: XXXX)...
```

Use the **LUN number** to identify your storage device.

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

### :pencil2: Provision VM in the cluster

Create a **new** content library while the data is stores in your shared datastore. 
Load the AlmaLinux ISO to the new content library, as done in the [previous tutorial](vsphere_intro.md).

> [!NOTE]
> To keep our vCenter clean, delete your old content library. 

Provision a new AlmaLinux in your cluster: 
- Choose your shared datastore to store your VM disk. 
- Allocate **only 1GB memory** for the VM.
- Note that you don't choose a specific host to provision the VM in, but let the DRS to schedule it for you. 
- Complete the OS installation setup. 

### :pencil2: Check the HA functionality

1. Open up a terminal in our lab and use the `virt-manager` command to shutdown the host in which your VM is running on. 
2. Observe the cluster in action. Does it detect the failure and restart the VM on another host?
   - Check if the VM is automatically powered on by another host.
   - Verify the failover time—how long does it take for the VM to become available again?
   - Look at the vSphere HA events and logs to confirm the failover process.
3. Turn on the host, and now let's **isolate the host** by disconnecting `vmk0` from the vSwitch.
   - Open the vSphere Client and navigate to the host where the VM is running on.
   - Go to **Networking** → **vSwitches**, locate `vmk0`, and disconnect it.

> [!NOTE]
> #### How vSphere Detects Isolation
> 1. The host checks for heartbeat signals from other hosts.
> 1. If no heartbeats are received, it tries to ping the isolation address.
> 1. If both fail, the host is declared isolated.

4. Observe how the cluster reacts:
   - Does HA detect the isolation?
   - Do the VMs remain running, or do they get restarted on another host?
   - Check the cluster events and tasks for any isolation-related events.
5. After a few minutes, reconnect `vmk0` and verify if the host rejoins the cluster properly.


### :pencil2: Host maintenance mode 

Maintenance Mode in an ESXi host is a special state that allows you to perform updates, repairs, or remove the host from the cluster, without disrupting the overall operation of your environment.

When a host is placed into Maintenance Mode:

- All running virtual machines (VMs) are migrated to other hosts in the cluster using vMotion.
- Features like DRS ensure VMs are automatically redistributed across other hosts in the cluster before the host enters Maintenance Mode.

1. Identify the host in which your VM is running on. Right-click on the host and select **Enter Maintenance Mode**.
2. Are all running VMs automatically migrated to other hosts using vMotion?
3. After successful migration, exit Maintenance Mode and verify that the host and VMs resume normal operation.

### :pencil2: HA Admission controller

In this exercise we'll see the Admission Controller in action. We will try to launch a VM in a cluster with poor available resources.

1. Power off your VMs.
2. Enter all hosts of the cluster to a maintenance mode, except one.
3. Try to turn on the VM. Intuitively, the VM should have been scheduled on the single functioning host of the cluster. 
   But the Admission Controller will prevent this due to insufficient resources to meet the VM's requirements.
4. Disable the admission controller, try to turn on the VM.



[vmware_storage_target_lun]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_storage_target_lun.png
[vmware_storage_shared_dstastore]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_storage_shared_dstastore.png
[vmware_storage_local_dstastore]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_storage_local_dstastore.png
[vmware_storage_iscsi_dstastore]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/vmware_storage_iscsi_dstastore.png
