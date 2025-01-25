# Intro to vCenter

The **vCenter Server** is a central point of management to manage resources from individual ESXi hosts, so that those resources can be shared among virtual machines in the entire
**datacenter**.

The **vSphere Web Client** is the interface to vCenter Server, lets you perform all administrative tasks by
using your browser.

## Entering the vShpere web client

1. From your lab environment, open the Chrome web browser and visit https://192.168.122
2. Login to the system as an administrator
3. Click on the **Menu** hamburger icon on the top left and select **Inventory**. This will take you to the inventory page where you find all the objects associated with vCenter, such as data centers, hosts, clusters, networking, storage, and virtual machines.

Before you can register your ESXi hosts to be managed by the vCenter Server, you have to create a datacenter. 

A **datacenter** in vCenter is a logical container used to group and manage IT resources such as hosts, clusters, virtual machines, networks, and storage. 

1. Right-click the top level object in the tree (the vCenter domain), then select **New Datacenter**, enter an appropriate name with your personal information, e.g. `john-us-east-1` and click **OK**.
2. Right-click on your datacenter, and choose **Add Host**.
3. Enter the IP address of your ESXi management interface and click **Next**.
4. Enter the root username and password for the ESXi host and click **Next**.
5. Complete the Add Host wizard by accepting the default values.

Time to create your first VM, based on the **AlmaLinux OS 9.5**.

> [!NOTE]
> Throughout the course we will be working with either CentOS or AlmaLinux. 

To launch the VM, we need the **AlmaLinux OS 9.5 Minimal ISO** image file.

vCenter allows you to store and manage ISOs and VM templates in a centralized repository called **Content Library**.

1. Navigate to **Menu** > **Content Libraries**.
2. In the main page, choose **Create**. The New Content Library wizard opens. 
3. On the **Name and location** page, enter a name, e.g. `john-images` and click **Next**.
4. On the **Configure content library** page, select the **Local content library**, and click **Next**.
5. No need to apply security policy right now (your content will be unencrypted and accessible by everyone in the vCenter).
6. Choose the main datastore to store you content. 
7. Finish the setup. 


Let's upload the AlmaLinux OS 9.5 Minimal ISO image into your content library. 

1. In the **Content Libraries** page, choose your content library. 
2. In the main page, click on **Actions** and **Import item**. 
3. In the **Import Library Item** wizard, choose the **URL** option and enter the public URL of AlmaLinux file: https://repo.almalinux.org/almalinux/9.5/isos/x86_64/AlmaLinux-9.5-x86_64-minimal.iso


# Exercises

### :pencil2: Create a virtual machine 

In this exercise you'll create you first (among many) AlmaLinux virtual machines. 

You can start by right-click on your datacenter, and choose **New Virtual Machine**.

Read the bellow notes and **carefully** follow the setup wizard:

- In the **Select a guest OS** page, select **Linux** as the OS family and **AlmaLinux (64 bit)** as the guest OS version. 
- In the **Customize Hardware** page, keep the default hardware configurations (1 CPU, 2 GB memory, 16GB disk), and load the ISO file from your content library as a CD/DVD device.
- You can always find answers in the [official docs](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vm_admin.doc/GUID-AE8AFBF1-75D1-4172-988C-378C35C9FAF2.html)

After creation, power on your instance, enter the web console, and follow the instructions to setup the OS.

> [!IMPORTANT]
> **When setup root password, enable the "Allow SSH login using password" option**
> 
> If you forgot to do that, login from the web console and perform:
> 
> ```bash
> echo 'PermitRootLogin yes' > /etc/ssh/sshd_config.d/01-permitroot.conf
> ```
> 
> Then restart your instance. 


Try to connect to your instance via SSH from the lab environment (outside vCenter) terminal:

```bash
ssh root@<your-instance-ip>
``` 

Change `<your-instance-ip>` to your instance IP. 

### :pencil2: VM Templates 

After you create a virtual machine, you can clone it to a template.

Templates are copies of virtual machines that let you create ready-for-use virtual machines.

1. Right-click on your VM instance, the choose **Clone**, and **Clone to template**.
2. Follow the cloning wizard. 
3. Finally, deploy another AlmaLinux VM from your template. 

