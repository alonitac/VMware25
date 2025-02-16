# Assignment: vSAN alignment and validation

You goal in this assignment is to has an up and running HA cluster with vSAN as the main datastore shared across hosts. 

To check the validity of your cluster, you'll deploy an application inside an AlmaLinux VM

## Guidelines 

### Provision the vSAN cluster

Follow the [vSphere cluster storage with vSAN](vcenter_storage.md) tutorial to create vSAN cluster with 3 hosts. 

#### Notes

- Your hosts must have **static** ip address. To check it, go to the **Configure** tab on your host, under **Networking** choose the **VMKernel adapter**, open the Edit Settings wizard of your adapter device, and unset **IPv4 Settings**, set a static ip. 
- At least one VMKernel that accept `vSAN`, `vMotion`, `Fault tolerance` and, `Management`. 
- HA and DRS should be turned on (unless you create the vSAN cluster). 
  All vCLS VMs should be running. In a cluster of 3 hosts, you should have 2-3 running vCLS VMs. 
  If you enter the **Monitor** tab of a given **Host** and you see an error like: 

  ```text
  Feature 'MWAIT' was 0, but must be 1. Failed to start the virtual machine. Module FeatureCompatLate power on failed. 
  ```
  
  This typically happens in nested ESXi environments where the physical CPU does not support or does not pass through certain advanced CPU features required by VMware.
  To fix this, see the related warning in the [HA Clusters](vsphere_ha_clusters.md) tutorial.
- When configuring vSAN, turn off HA and DRS capabilities.
- If you vSAN cluster is in unhealthy state and you don't know how to address the issue, 
  it's always possible to shutdown your ESXi hosts, re-run the `add-vsan-disks ESXI_HOST_NAME` command, and reallocate the vSAN disks.  

#### Troubleshooting 

- The old good way can always work: shutdown your ESXi and turn on again. 
- Take a look on the tasks and events under the **Monitor** tab of the object your debug. 
- The **Summary** tab of a given VM can help you answer questions like "Which datastore or host was the VM configured on?"
- The **Summary** tab of a given host/cluster can help you check for any warnings or errors related to hardware or software. 
- To see the disk healthy of your 
- For advanced vSAN troubleshooting, you can start the SSH service on your host, and SSH into your host by running the below command in a terminal from our lab:

  ```bash
  ssh root@MY_HOST_IP
  ```
  
  While changing `MY_HOST_IP` by your real IP. 
  
  These commands may helo you debug your vSAN cluster:
   - General information about the vSAN cluster: `esxcli vsan cluster get`.
   - Remove host from vSAN cluster: `esxcli vsan storage remove --host <host_ip>`.
   - List disk status per host: `esxcli vsan storage list`.
   - List all cluster disks and status: `esxcli vsan debug disk list`.

## Validate your vSAN cluster

To validate the reliability of your cluster, you will deploy two AlmaLinux VMs.
One will act as the **client** and the other as the **server**.

The client will communicate with the server while you simulate a host failure.
Your goal is to measure the downtime experienced by the server during this failure.

#### Steps

1. Deploy two AlmaLinux VMs in your cluster, ensuring they are both running and accessible.

   If both VMs were scheduled on the same host by the DRS, migrate one of them to a different host (right-click -> migrate).
   They must run on separate hosts for the simulation.
2. SSH into the first VM (preferably from a local terminal in our lab RDP connection).
3. On the first VM, install Docker by running the following commands (don't worry if you don't familiar with Docker containers yet, this content will be covered soon):

   ```
   sudo dnf -y install dnf-plugins-core
   sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   sudo systemctl enable --now docker
   ```

4. Start the server application (a "Netflix" frontend website) by running:

   ```bash
   sudo docker run -p 3000:3000 alonithuji/netflix-frontend:0.0.1
   ```

5. On the second VM, install Docker again and run the following command to continuously send requests to the server:

   ```bash
   docker run --rm alpine:latest /bin/sh -c "while true; do echo \$(date) && wget -q --spider http://YOUR_SERVER_VM_IP:3000; sleep 3; done"
   ```
   
   Replace `YOUR_SERVER_VM_IP` with the IP address of the server VM.

6. Simulate a host failure by stopping or disconnecting the server VM.
   Measure the downtime by checking the timestamps of when the client can no longer reach the server.


## Good Luck

