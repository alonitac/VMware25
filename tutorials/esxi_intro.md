# Intro to ESXi

An **ESXi host** is a hypervisor developed by VMware that enables you to run and manage multiple virtual machines (VMs) on a single physical server.

## Installing an ESXi

Usually, a fresh ESXi 8 installation is done by directly installing it on a physical server using an ISO file and configuring the boot settings. 
However, since we are operating in a lab environment with **nested ESXi** (ESXi running as a VM on Ubuntu), we will start the installation from setup wizard, as if you already booted from the installation media.


You'll be installing ESXi on a system equipped with a **4-core Intel Xeon processor, 4GB RAM**.

in addition, you have 2 disks, and 2 uplinks... 

1. Connect to your lab environment via RDP.
2. To view the **Direct Console User Interface (DCUI)** screen of your ESXi host (as if you were physically in front of it), open a terminal and type the `virt-manager` command.
3. In the opened window, select your ESXi host and click **Open**. 
4. Once the ESXi installer is launched, follow the instructions there to complete the host setup wizard. **Notes**:
   - From the list of available disks, **you should choose the `32GB` disk**. The other disk will be used by the VMs running on this host.
   - Choose **easy to remember** root account password (don't worry you'll never run your production workloads on this host).

> [!TIP]
> - Press **Ctrl + Alt** together to release the mouse from the ESXi screen view.


5. Once the installation compete, click **Enter** to reboot your host. 
6. Since this is a virtual ESXi (not a real physical server as in real life), you have to start it manually after the reboot by (change `<host-name>` to your host name).:
 
```bash
sudo virsh start <host-name>
```


## Configure the management network

We will use again the **Direct Console User Interface (DCUI)** to configure the Network Interface Cards (NICs) of our host.

A NIC is hardware that connects a device to a network. It's recommended to have at least 2 NICs in order to provide **redundancy**. 


1. Once you ESXi host is running, use again the `virt-manager` command to view the DCUI.
2. From the main DCUI screen, press the **F2** key to login.
3. Enter your password for the `root` user. 
4. At the **System customization** screen, navigate to **Configure Management Network**, then to **Network Adapters**.
5. Select the 2 adapters to be available in your host:
   - [x] `vmnic0`
   - [x] `vmnic1`
6. Next, enter the **IPv4 Configuration** screen, and choose **Set static IPv4 address**. The is **no need** to change the IP address of your host. 
7. Exit the configurations screen and choose **Yes** when asking to apply the configurations. 
8. In the next step, choose **Test Management Network** to ensure your setting were configured properly. Make sure you ping the **default gateway**, the **primary DNS server** (should be the same), and the **ESXi hostname** (should be failed).


#### 🧐 Check yourself

Exploring the **Configure Management Network** screen, answer the below questions:

- What is the IPv4 address of your host?
- Where did it get the IP address from? 
- Which IP address that my packets would be sent to, when I want to talk with the Internet from my ESXi host?


## Working with the ESXi Host Client

The VMware Host Client is a web-based client that is used to connect to and manage single ESXi hosts.

You can use the VMware Host Client to perform administrative tasks on your target ESXi host.


1. Open up the Chrome browser and visit `https://<my-host-ip>` (while `<my-host-ip>` is your ESXi host IPv4).

> [!TIP]
> Save this address in your favorites. 

As can be seen, here you configure setting about your CPU, memory, and storage, as well as access logs for troubleshooting.


The next step is to create a **Datastore** to store VMs disks.

In our case, your ESXi host has 2 physical disks, one of 32GB for ESXi's internal storage and the other with 100GB disk exclusively for VM storage. 

Datastores are storage containers used to store virtual machine VM disks, snapshots, and ISO images.
They abstract the physical storage.

1. Click **Storage** in the VMware Host Client inventory, under **Datastores** tab, choose **New datastore**.
2. The **New datastore** wizard opens. On the **Select creation type** page, select **Create new VMFS datastore** and click **Next**. 
3. On the **Select device** page: 

   - Enter a name for the new datastore. E.g. `VM_storage`.
   - Select the 100GB disk device.
   - Click **Next**.
4. On the **Select partitioning options** page, choose **Use full disk** to allocate the whole physical disk. 
5. On the **Ready to complete** page, review the configuration details and click **Finish**. 


> [!NOTE]
> When vCenter Server is present, it is the preferred option to use to manage your ESXi hosts.
> It offers much more management options, and allows you to manage multiple ESXi hosts altogether. 
> 
> Using the Host client is useful in test/dev environments or when the vCenter Server is not reachable.

