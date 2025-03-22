# Provision 2 nodes Kubernetes cluster in vSphere using kubeadm


In this section you'll provision a Kubernetes cluster consists by **one control plane node** and **one worker node**.
Although vSphere offers Tanzu, a managed service that simplifies cluster setup and management, we will manually set up the cluster using `kubeadm`.
This hands-on approach will allow you to gain a deeper understanding of Kubernetes architecture and the processes involved in cluster creation and configuration. 

The underlying nodes infrastructure are based on - you guess right - **Virtual machines**. Let's get started. 

### Prepare infrastructure 

1. Launch two AlmaLinux with 2 CPUs, 4GB, 30GB disk. Naming them `<your-name>-control-plane` and `<your-name>-worker`.
2. Execute the below script **in both machines**. 

```bash
# These instructions are for Kubernetes v1.32.
KUBERNETES_VERSION=v1.32
CRIO_VERSION=v1.32

# Enable IPv4 packet forwarding. sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

cat <<EOF | tee /etc/yum.repos.d/cri-o.repo
[cri-o]
name=CRI-O
baseurl=https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/rpm/
enabled=1
gpgcheck=1
gpgkey=https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/rpm/repodata/repomd.xml.key
EOF

sudo yum install -y cri-o kubelet kubeadm kubectl --disableexcludes=kubernetes

# Start the CRI-O container runtime and kubelet
sudo systemctl enable --now kubelet
sudo systemctl enable --now crio.service
sudo systemctl enable --now kubelet
sudo systemctl stop firewalld
sudo systemctl disable firewalld

# Disable swap memory
sudo swapoff -a

# Add the command to crontab to make it persistent across reboots
(crontab -l ; echo "@reboot /sbin/swapoff -a") | crontab -
```

The script essentially installs: 

- `kubeadm` - the [official Kubernetes tool](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) that initializes the cluster and make it up and running.
- `kubelet` - a Linux service that runs on each node in the cluster, responsible for Pods lifecycle.
- `cri-o` as the container runtime.
- `kubectl` - to control the cluster.

### Initialize the cluster 

1. From the **control-plane node**, initialize the cluster by:

``bash
sudo kubeadm init
``

2. **Carefully** read the output to understand how to start using your cluster, and how to join the worker node to be part of the cluster. 

   The `kubeadm init` command essentially does the below:
   
   1. Runs pre-flight checks.
   2. Creates certificates that used by different components for secure communication.
   3. Generates `kubeconfig` files for cluster administration.
   4. Deploys the `etcd` db as a Pod.
   5. Deploys the control plane components as Pods (`apiserver`, `controller-manager`, `scheduler`).
   6. Starts the `kubelet` as a Linux service.
   7. Install addons in the cluster (`coredns` and `kube-proxy`).

   ![][k8s_architecture_kubeadm]

   Make sure you understand the [role of each component in the architecture](https://kubernetes.io/docs/concepts/architecture/).


> [!NOTE]
> - To run `kubeadm init` again, you must first [tear down the cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#tear-down).
> - The join token is valid for 24 hours. Read here [how to generate a new one](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes) if needed.
> - For more information about initializing a cluster using `kubeadm`, [read the official docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/). 

3. Make sure you have two (not yet ready) nodes cluster by executing from the control-plane machine:

```bash
kubectl get nodes
```

### Install Pod networking plugin

When deploying a Kubernetes cluster, there are 2 layers of networking communication:

![][k8s_cni]

- Communication between Nodes (denoted by the green line). This is managed for us **by the dSwitch**.
- Communication between Pods (denoted by the purple line). Although the communication is done on top of the dSwitch, communication between pods using their own cluster-internal IP address is **not** implemented for us by vSphere.   

You must deploy a [Container Network Interface (CNI)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) network add-on so that your Pods can communicate with each other. 
There are many [addons that implement the CNI](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy). 

We'll install [Calico](https://docs.tigera.io/calico/latest/about/), simply by:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/calico.yaml
```

Make sure you have nodes are ready:

```bash
kubectl get nodes
```

[k8s_architecture_kubeadm]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/k8s_architecture_kubeadm.png
[k8s_cni]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/k8s_cni.png