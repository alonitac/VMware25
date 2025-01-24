# Intro to ESXi

An **ESXi host** is a hypervisor developed by VMware that enables you to run and manage multiple virtual machines (VMs) on a single physical server.

## Installing an ESXi

Usually, a fresh ESXi 8 installation is done by directly installing it on a physical server using an ISO file and configuring the boot settings. 
However, since we are operating in a lab environment with **nested ESXi** (ESXi running as a VM on Ubuntu), we will start the installation from setup wizard, as if you already booted from the installation media.


You'll be installing ESXi on a system equipped with a **4-core Intel Xeon processor, 4GB RAM**.

in addition, you have 2 disks, and 2 uplinks... 

1. Connect to your lab environment via RDP.
2. To view the **Direct Console User Interface (DCUI)** screen of your ESXi host (as if you were physically in front of it), open a terminal and type the `virt-viewer` command.
3. In the **Choose a virtual machine** window, select your ESXi host and click **Connect**. 
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


1. Once you ESXi host is running, use again the `virt-viewer` command to view the DCUI.
2. From the main DCUI screen, press the **F2** key to login.
3. Enter your password for the `root` user. 
4. At the **System customization** screen, navigate to **Configure Management Network**, then to **Network Adapters**.
5. Select the 2 adapters to be available in your host:
   - [x] `vmnic0`
   - [x] `vmnic1`

#### 🧐 Check yourself

Exploring the **Configure Management Network** screen, answer the below questions:

- What is the IPv4 address of your host?
- Where did it get the IP address from? 
- Which IP address that my packets would be sent to, when I want to talk with the Internet from my ESXi host?



   










While ESXi is a specialized operating system designed for virtualization, it has a minimalistic structure and is not a full-fledged OS like Linux or Windows. Here's a breakdown:
