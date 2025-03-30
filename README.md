<div align="center">

# Modern IT Infrastructure - VMware and Kubernetes


</div>

Welcome to the main repository for the VMware and Kubernetes course!
This is your place for all course content, exercises, and tutorials.

Work hard, and have fun 😊


## Onboard the course 

No any installation required. 
All you lab environment is available through an RDP connection (access will be provided at class).


## Studying Guide

<table width="100%">
<tr><th>#</th><th>Topic</th><th>Tutorial</th><th colspan="3">&nbsp;&nbsp;&nbsp;Resources&nbsp;&nbsp;&nbsp;</th><th>Project</th><th>Recording</th></tr>

<tr>
 <td align="center" colspan="8"><br><b>Intro to Virtualization in VMware</b><br>Install and configure a single ESXi host, standard switch network topology, provision VMs via the vSphere web client<br><br></td>
</tr>

<tr>
 <td>1</td>
 <td>Intro</td>
 <td>Intro to virtualization in VMware</td>
 <td><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/slides/vmware_virtualization_intro.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/slides.png" /></a></td>
 <td></td>
 <td></td>
 <td></td>
 <td><code>recordings/2025-01-26</code></td>
</tr>	

<tr>
 <td>2</td>
 <td>VMware</td>
 <td><a href="tutorials/esxi_intro.md">ESXi installation and configuration</a></td>
 <td></td>
 <td></td>
 <td></td>
 <td></td>
 <td></td>
</tr>

<tr>
 <td>3</td>
 <td>VMware</td>
 <td><a href="tutorials/esxi_networking.md">The networking stack in a single ESXi host</a></td>
 <td><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/slides/esxi_networking.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/slides.png" /></a></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/multichoice-questions/vmwre_esxi_intro.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/qm.png" /></a></td>
 <td align="center"><a href="tutorials/esxi_networking.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td></td>
 <td></td>
</tr>

<tr>
 <td>4</td>
 <td>VMware</td>
 <td><a href="tutorials/vsphere_intro.md">Create a virtual machine via the vSphere client</a></td>
 <td></td>
 <td></td>
 <td><a href="tutorials/vsphere_intro.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td></td>
 <td></td>
</tr>

<tr>
 <td>5</td>
 <td>Networking</td>
 <td><a href="tutorials/networking_OSI_model.md">The OSI model</a></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/slides/vmware_networking_OSI_model.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/slides.png" /></a></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/multichoice-questions/networking_OSI_model.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/qm.png" /></a></td>
 <td align="center"><a href="tutorials/networking_OSI_model.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td><code>recordings/2025-02-02</code></td>
</tr>

<tr>
 <td align="center" colspan="8"><br><b>Host clusters</b><br>Create cluster of multiple ESXi hosts<br><br></td>
</tr>


<tr>
 <td>6</td>
 <td>VMware</td>
 <td><a href="tutorials/vsphere_ha_clusters.md">HA host clusters</a></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/slides/vmware_ha_cluster.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/slides.png" /></a></td>
 <td></td>
 <td align="center"><a href="tutorials/vsphere_ha_clusters.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td></td>
 <td></td>
</tr>

<tr>
 <td>8</td>
 <td>VMware</td>
 <td><a href="tutorials/vcenter_networking.md">Distributed switches in HA cluster</a></td>
 <td></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/multichoice-questions/vmware_cluster_networking.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/qm.png" /></a></td>
 <td align="center"><a href="tutorials/vcenter_networking.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td></td>
 <td></td>
</tr>

<tr>
 <td>9</td>
 <td>VMware</td>
 <td><a href="tutorials/vcenter_storage.md">Cluster storage with vSAN</a></td>
 <td></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/multichoice-questions/vmware_cluster_vsan.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/qm.png" /></a></td>
 <td align="center"><a href="tutorials/vcenter_storage.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td></td>
 <td></td>
</tr>

<tr>
 <td>10</td>
 <td>VMware</td>
 <td><a href="tutorials/vsan_cluster_summary_assignment.md">vSAN cluster summary assignment</a></td>
 <td></td>
 <td></td>
 <td></td>
 <td></td>
 <td></td>
</tr>

<tr>
 <td align="center" colspan="8"><br><b>Linux Administration</b><br><br></td>
</tr>

<tr>
 <td>11</td>
 <td>Linux</td>
 <td><a href="tutorials/provision_alma.md">Provision AlmaLinux VM</a></td>
 <td align="center"></td>
 <td align="center"></td>
 <td align="center"></td>
 <td align="center"></td>
 <td></td>
</tr>

<tr>
 <td>12</td>
 <td>Linux</td>
 <td><a href="tutorials/linux_intro.md">Intro to Linux</a></td>
 <td align="center"></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/multichoice-questions/vmware_linux_intro.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/qm.png" /></a></td>
 <td align="center"><a href="tutorials/linux_intro.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td></td>
</tr>

<tr>
 <td>13</td>
 <td>Linux</td>
 <td><a href="tutorials/linux_file_management.md">File management</a></td>
 <td align="center"></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/multichoice-questions/vmware_linux_file_management.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/qm.png" /></a></td>
 <td align="center"><a href="tutorials/linux_file_management.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td></td>
</tr>


<tr>
 <td>14</td>
 <td>Linux</td>
 <td><a href="tutorials/linux_processes.md">Processes</a></td>
 <td align="center"></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/multichoice-questions/vmware_linux_processes.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/qm.png" /></a></td>
 <td align="center"><a href="tutorials/linux_processes.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td></td>
</tr>

<tr>
 <td align="center" colspan="8"><br><b>Containers and Orchestration</b><br><br></td>
</tr>

<tr>
 <td>15</td>
 <td>Containers</td>
 <td><a href="tutorials/docker_intro.md">Intro to Docker containers</a></td>
 <td><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/slides/docker_intro_non_devops.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/slides.png" /></a></td>
 <td align="center"></td>
 <td align="center"></td>
 <td></td>
</tr>

<tr>
 <td>16</td>
 <td>Containers</td>
 <td><a href="tutorials/docker_containers.md">Working with containers</a></td>
 <td align="center"></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/multichoice-questions/docker_containers_non_devops.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/qm.png" /></a></td>
 <td align="center"><a href="tutorials/docker_containers.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td></td>
</tr>

<tr>
 <td>17</td>
 <td>Containers</td>
 <td><a href="tutorials/docker_networking.md">Docker networking</a></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/slides/docker_networking.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/slides.png" /></a></td>
 <td align="center"><a target="_blank" href="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/multichoice-questions/docker_networking_non_devops.html"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/qm.png" /></a></td>
 <td align="center"><a href="tutorials/docker_networking.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td></td>
</tr>

<tr>
 <td>18</td>
 <td>Containers</td>
 <td><a href="tutorials/docker_volumes.md">Docker volumes</a></td>
 <td align="center"></td>
 <td align="center"></td>
 <td align="center"><a href="tutorials/docker_volumes.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td></td>
</tr>

<tr>
 <td>19</td>
 <td>Containers</td>
 <td><a href="tutorials/docker_compose.md">Docker compose</a></td>
 <td align="center"></td>
 <td align="center"></td>
 <td align="center"><a href="tutorials/docker_compose.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td></td>
</tr>


<tr>
 <td>20</td>
 <td>Orchestration</td>
 <td><a href="tutorials/k8s_provision_cluster_kubeadm_aws.md">Provision Kubernetes cluster in vCenter</a></td>
 <td align="center"></td>
 <td align="center"></td>
 <td align="center"></td>
 <td align="center"></td>
 <td></td>
</tr>

<tr>
 <td>21</td>
 <td>Orchestration</td>
 <td><a href="tutorials/k8s_intro.md">k8s intro</a></td>
 <td align="center"></td>
 <td align="center"></td>
 <td align="center"><a href="tutorials/k8s_intro.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td></td>
</tr>

<tr>
 <td>22</td>
 <td>Orchestration</td>
 <td><a href="tutorials/k8s_core_workloads.md">k8s core workloads</a></td>
 <td align="center"></td>
 <td align="center"></td>
 <td align="center"><a href="tutorials/k8s_core_workloads.md#exercises"><img src="https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/pen.png" /></a></td>
 <td align="center"></td>
 <td></td>
</tr>

</table>
