# Intro to Active Directory Environment

In the following tutorial, we will create a AD domain composed by 2 computer.


## Launch a Windows Server virtual machine in AWS

Amazon EC2 (Elastic Compute Cloud) is a web service that provides resizable compute capacity in the cloud. 
It allows users to create and manage virtual machines, commonly referred to as "instances", which can be launched in a matter of minutes and configured with custom hardware, network settings, and operating systems.


1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/).

2. From the EC2 console dashboard, in the **Launch instance** box, choose **Launch instance**, and then choose **Launch instance** from the options that appear\.

3. Under **Name and tags**, for **Name**, enter a descriptive name for your instance\.

4. Under **Application and OS Images \(Amazon Machine Image\)**, do the following:

   1. Choose **Quick Start**, and then choose **Windows**\. This is the operating system \(OS\) for your instance\.
   
5. Under **Instance type**, from the **Instance type** list, you can select the hardware configuration for your instance\. Choose the `t3.medium` instance type.

6. Under **Key pair \(login\)**, choose **create new key pair** the key pair that you created when getting set up\.

   1. For **Name**, enter a descriptive name for the key pair\. Amazon EC2 associates the public key with the name that you specify as the key name\.

   2. For **Key pair type**, choose either **RSA**.

   3. For **Private key file format**, choose the format in which to save the private key\. Choose **pem** for key type.

   4. Choose **Create key pair**\.
   
     **Important**  
     This step should be done once! once you've created a key-pair, use it for every EC2 instance you are launching. 

   5. The private key file is automatically downloaded by your browser\. The base file name is the name you specified as the name of your key pair, and the file name extension is determined by the file format you chose\. Save the private key file in a **safe place**\.
      
      **Important**  
      This is the only chance for you to save the private key file\.

7. In the **Network Settings** section, under **Firewall (security groups)**, choose **Select existing security group**, and choose the **default** one. 
8. Keep the default selections for the other configuration settings for your instance\.

9. Review a summary of your instance configuration in the **Summary** panel, and when you're ready, choose **Launch instance**\.

10. A confirmation page lets you know that your instance is launching\. Choose **View all instances** to close the confirmation page and return to the console\.

11. On the **Instances** screen, you can view the status of the launch\. It takes a short time for an instance to launch\. When you launch an instance, its initial state is `pending`\. After the instance starts, its state changes to `running` and it receives a public DNS name\.

12. It can take a few minutes for the instance to be ready for you to connect to it\. Check that your instance has passed its status checks; you can view this information in the **Status check** column\.

> [!NOTE]
> When stopping the instance, please note that the public IP address may change, while the private IP address remains unchanged


To connect to your instance:

1. Choose you instance ID in the On the **Instances** screen.
2. Click on the **Connect** button, and choose the **RDP client** option. 
3. Follow the instructions there. 



## Configuring the Domain Controller and Installing Active Directory

### When setting up a domain controller in a production environment, it's good idea to have your own computer name: 

1. From the windows start button, go to **Server Manager**.
2. Choose **Local Server**, click on your **Computer Name**.
3. In the opened dialog, click **Change**, then change computer name to `DC01`.
4. Click **OK**

[//]: # (### Configure your domain controller instance with the static IP address:)

[//]: # ()
[//]: # (1. Open the **Control Panel**, Click on **Network and Internet**, then **Network and Sharing Center**.)

[//]: # (2. Click on the available `Ethernet` connection. )

[//]: # (3. In the opened dialog, choose **Properties**, then in the items list, choose **Internet Protocol Version 4 &#40;TCP/IPv4&#41;** and click **Properties**.)

[//]: # (4. Choose the **Use the following IP Address** button and fill in the information according to your instance configurations &#40;can be found using `ifconfig` command&#41;.)

[//]: # ()

### Installing Active Directory Services

1. Open **Server Manager** and choose **Add Roles and Features**.
2. In the opened wizard, under **Server Roles**, choose **Active Directory Domain Services** and **DNS Server**.
3. Follow the default configurations to install these roles. 

Let the installation complete...

4. After installation, click on the warning flag in Server Manager and select **Promote this server to a domain controller**.
5. Select **Add a new forest**, then specify the **root domain** DNS name e.g. `corp.mycorp.com`.
6. Choose a password, and continue to the end of the configuration wizard with the default values. 
7. When done, restart the server.


## Create objects in you AD domain 

Let's create organizational units for Users, Groups, and Computers. 

1. In the **Server Manager**, click on **Tool** in the top navigation menu, and choose **Open Active Directory Administrative Center**.
2. Right-click on your domain, select **New** > **Organizational Unit**.
3. Name it `production` to represent your production environment where all your business systems, applications, and resources reside.
4. Under the `production` OU, create 3 more OUs for `Users`, `Groups` and `Computers`. 

You can open **Active Directory Users and Computers** (under the **Tools** menu) to see a hierarchical structure of your OUs.

### Create Users

We now want to create some user accounts to represent employee in our organization.

These accounts will allow individuals to authenticate to the domain and access resources based on their assigned permissions.

1. Right-click on the `Users` OU, select **New** > **User**.
2. Enter user details and set a password.

We would like to assign the created user to be the **domain admin**. 

3. Right-click on the created user object, and choose **Add to group**.
4. Type in `Domain Admins` the object name box (you can validate your group name by **Check names** button). 
    This group is a built-in security group that has complete administrative control over the entire domain.
5. Create one more user.
6. Create a group named `developers` and add the created user to the group. 

### Add machines to the domain

1. Create a new Windows VM from the AWS console. 
2. Connect to the VM using RDP.

When a computer joins a domain, it needs to locate domain resources, especially the domain controller.
This happens through specific DNS records that the domain controller creates and maintains.
Hence, we have to configure our server to use the **domain controller** as the DNS server. 

1. Open the **Control Panel**, Click on **Network and Internet**, then **Network and Sharing Center**.
2. Click on the available `Ethernet` connection. 
3. In the opened dialog, choose **Properties**, then in the items list, choose **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
4. Choose the **Use the following DNS server Address** button and under **Preferred DNS server**, specify the (private) ip address of your domain controller (if you don't remember, the address can be found in the instance info page in AWS console)


Now let's change the server name and add it to the domain.

1. From the windows start button, go to **Server Manager**.
2. Choose **Local Server**, click on your **Computer Name**.
3. In the opened dialog, click **Change**, then change computer name to `DC-1`, and specify your domain DNS under **Member of:** -> **Domain**.
4. You will be asked to log in, enter the username and password of the user belonging to the `developers` group. 

Once the instance has joined the domain, the `Administrator` user you use to connect to the instance would be able to do so anymore, since he is not part of the domain.

To allow your domain user to access via RDP:

1. Search **Remote Desktop Settings** and open it.
2. Choose **Remote desktop users**.
3. In the opened dialog, click **Add**, and enter your domain user. validate with **Check names**.
4. Authenticate yourself and click **Ok**.

Now you can safely restart the instance. 

Lastly, we want to move the new computer account to the appropriate OU.

1. From your domain controller instance, go to the **Server Manager**, then **Tools**, and **Active Directory Users and Computers**.
2. You should find your computer under the **Computer** OU in you domain, right-click on it, and choose **Move...**. 
3. Choose the **Computers** OU under the `production` OU you've created. 

## Deploy software to your domain computers

Now let's deploy an app (PuTTY) to our domain computers using **group policy**.
Group policy is a feature that allows administrators to centrally manage and configure Windows environments, applying settings to multiple computers automatically without having to configure each machine individually.

First, let's run the following PowerShell script to download the installer into `C:\Software\NotepadPP\NotepadPlusPlus.exe`:

```powershell
# Create directory if it doesn't exist
New-Item -Path "C:\Software\Putty" -ItemType Directory -Force

# Set the download URL for Putty MSI (64-bit version)
$url = "https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.83-installer.msi"

# Set destination path
$destination = "C:\Software\Putty\putty.msi"

# Download the file
Invoke-WebRequest -Uri $url -OutFile $destination

# Verify the download
if (Test-Path -Path $destination) {
    Write-Host "Download completed successfully: $destination"
    # Get file info
    Get-Item $destination | Select-Object Name, Length, LastWriteTime
} else {
    Write-Host "Download failed!" -ForegroundColor Red
}

```

Let's make the folder contains the installation sharable:

1. Right-click on the `Software` folder.
1. Select **Properties**.
1. Click on the **Sharing** tab
1. Click **Advanced Sharing**.
1. Check the box for **Share this folder**
1. Click **Permissions** and make sure **Everyone** is contained, and read is allowed.
1. Click **OK** on all dialog boxes.

Let's create the group policy:

1. In the server manager, **Tools** menu, open **Group Policy Management** Console.
1. Right-click on the OU containing your computer.
1. Select **Create a GPO in this domain, and Link it here**.
1. Name it something descriptive like **Deploy client app**
1. Click **OK**.


1. Right-click on your new GPO and select **Edit**.
2. Under **Computer Configuration**, navigate to **Policies** > **Software Settings** > **Software installation**.
3. Right-click in the right pane and select **New** > **Package**.
4. Browse to the network share using the UNC path: `\\DC01\Software\Putty\putty.msi`
5. When prompted for deployment method, select **Assigned** (means the software will be installed automatically without user intervention).
6. Click **OK**.

Restart the other computer in the domain, make sure PuTTY is available. 


