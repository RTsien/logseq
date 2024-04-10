title:: DIY: Create Your Own Cloud With Kubernetes (highlights)
author:: [[Kubernetes – Production-Grade Container Orchestration]]
full-title:: "DIY: Create Your Own Cloud With Kubernetes"
category:: #articles
url:: https://kubernetes.io/blog/2024/04/05/diy-create-your-own-cloud-with-kubernetes-part-2/
summary:: Author: Andrei Kvapil (Ænix)
Continuing our series of posts on how to build your own cloud using just the Kubernetes ecosystem.
In the previous article, we
explained how we prepare a basic Kubernetes distribution based on Talos Linux and Flux CD.
In this article, we'll show you a few various virtualization technologies in Kubernetes and prepare
everything need to run virtual machines in Kubernetes, primarily storage and networking.
We will talk about technologies such as KubeVirt, LINSTOR, and Kube-OVN.
But first, let's explain what virtual machines are needed for, and why can't you just use docker
containers for building cloud?
The reason is that containers do not provide a sufficient level of isolation.
Although the situation improves year by year, we often encounter vulnerabilities that allow
escaping the container sandbox and elevating privileges in the system.
On the other hand, Kubernetes was not originally designed to be a multi-tenant system, meaning
the basic usage pattern involves creating a separate K...
![](https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.png)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- **Kata Containers** implements the CRI (Container Runtime Interface) and provides an additional level of isolation for standard containers by running them in virtual machines. But they work in a same single Kubernetes-cluster.
	  
	  ![](https://kubernetes.io/blog/2024/04/05/diy-create-your-own-cloud-with-kubernetes-part-2/kata-containers.svg)
	  
	  A diagram showing how container isolation is ensured by running containers in virtual machines with Kata Containers ([View Highlight](https://read.readwise.io/read/01hv1vxpgqn87qehrwng4843as))
	- **KubeVirt** allows running traditional virtual machines using the Kubernetes API. KubeVirt virtual machines are run as regular linux processes in containers. In other words, in KubeVirt, a container is used as a sandbox for running virtual machine (QEMU) processes. This can be clearly seen in the figure below, by looking at how live migration of virtual machines is implemented in KubeVirt. When migration is needed, the virtual machine moves from one container to another.
	  
	  ![](https://kubernetes.io/blog/2024/04/05/diy-create-your-own-cloud-with-kubernetes-part-2/kubevirt-migration.svg)
	  
	  A diagram showing live migration of a virtual machine from one container to another in KubeVirt ([View Highlight](https://read.readwise.io/read/01hv1vxtrw3x5pf1sq9c2jnwym))
	- Virtual machines with containerDisk are well suited for creating Kubernetes worker nodes and other VMs that do not require state persistence. ([View Highlight](https://read.readwise.io/read/01hv1w2jjxrgwbqg4w55ffpgxe))