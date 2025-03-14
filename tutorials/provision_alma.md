# Provision AlmaLinux VM

Before we'll delve into Linux administration topics, make sure you have a single AlmaLinux VM up and running. 

### Notes 

- Allocate 4GB memory.
- Don't forget to **enable networking** as part of the installation process.
- The VM should reside in your HA cluster.  

### Access the VM

You can access the VM in 2 ways:

1. (Recommended) Connect via SSH from your local machine to the lab machine, by: `ssh USERNAME@lab.exit-zero.click` (while changing `USERNAME` to your user). 
   Then, from the lab terminal, SSH again to your VM: `ssh root@YOUR_VM_IP`. This method is known as **SSH jump host**.
2. Connect via SSH from the lab RDP terminal session: `ssh root@YOUR_VM_IP`

