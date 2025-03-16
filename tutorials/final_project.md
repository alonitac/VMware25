# Final Assessment

This assignment reflects the knowledge and skills you’ve gained throughout the course.

Some of you may find it easy, while others may find it challenging.

You will set up an HA and vSAN cluster with three hosts, provision multiple virtual machines, and deploy an application within them.

## Guidelines

### Provision infrastructure

- Create 3 ESXi hosts, use the `create-esxi <host-name>` command (while changing `<host-name>` to your hostname), to create a 4CPU and 16GB host with 3 disks as follows:
  - 32GB - to be used by the ESXi OS.
  - 100GB SSD - to be used later by the cache tier of you vSAN cluster
  - 1TB - to be used later by the capacity tier of you vSAN cluster.

- Connect your hosts to the vCenter and create your Data Center.
- Create a HA cluster and add you hosts:
   - It's recommended to start the cluster when HA, DRS, and vSAN are disabled, and enable them later on once the cluster is properly configured.
   - In the `vmk0` configuration of each host of your cluster, set the host IP **to be static**. Also, don't forget to enable desired capability (e.g. vMotion).
   - You can ignore "Warning: no datastore have been configured". Soon you'll configure the vSAN as the **only** datastore in your cluster. 

- Create Distributed Switch with at least 2 groups: 
   - Group for VMs NICs.
   - Group for `vmk0`s management NICs.
   
- Migrate your hosts to the dSwitch. 
- Configure a vSAN cluster:
   - **Don't** enable ENA.
   - Configure the 100GB SSD disk to be the cache tier.
   - Configure the 1TB disk in each host to be the capacity tier. 
   - Make sure your vSAN cluster consist by a single Disk Group (usually named `Group 1`).
   - If you check in health of your cluster (in the skyline health page), the only you should see is `SCSI controller is not certified`. Other than that your cluster should have no issues.
   - You should not configure any other data store in your cluster, except the vSAN shared datastore. 

- Make sure all `vCLS` VMs are running.
- Enable DRS and HA.
- Create a Content Library with the [AlmaLinux OS 9.5 Minimal ISO](https://repo.almalinux.org/almalinux/9.5/isos/x86_64/AlmaLinux-9.5-x86_64-minimal.iso) image stored in it.
- Create 2 AlmaLinux:
  - 2 CPU, 4GB, 30GB disk
  - Connected to the dSwitch network.
  - When installing it, make sure network is enabled.
  - You can install only 1 machine and clone it to create other VMs easily.


> [!IMPORTANT]
> #### Make sure you know to answer the following questions:
> 
> - What is the purpose of the 3 different disk sizes in your ESXi configuration?
> - What will happen in the IP of your host will change?
> - What capabilities did you enabled on each **vmkernel interface**? why? 
> - What are the advantages of using a **Distributed Switch** over a **Standard Switch**?
> - What is the name of your VMs and management **port groups**?
> - What is the maximum number of ports that can be configured in your dSwitch?
> - What happen if you disconnect one of your hosts? (do it!).
> - What happens if a vCLS VM stops running?
> - What is a vSAN disk group and why is your cluster configured with a single disk group?
> - What is the primary purpose of HA? and of DRS?



### Deploy application 

TBD

# Good Luck
