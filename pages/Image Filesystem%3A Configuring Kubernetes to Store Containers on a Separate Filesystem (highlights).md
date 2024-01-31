title:: Image Filesystem: Configuring Kubernetes to Store Containers on a Separate Filesystem (highlights)
author:: [[Kubernetes – Production-Grade Container Orchestration]]
full-title:: "Image Filesystem: Configuring Kubernetes to Store Containers on a Separate Filesystem"
category:: #articles
url:: https://kubernetes.io/blog/2024/01/23/kubernetes-separate-image-filesystem/
summary:: Author: Kevin Hannon (Red Hat)
A common issue in running/operating Kubernetes clusters is running out of disk space.
When the node is provisioned, you should aim to have a good amount of storage space for your container images and running containers.
The container runtime usually writes to /var.
This can be located as a separate partition or on the root filesystem.
CRI-O, by default, writes its containers and images to /var/lib/containers, while containerd writes its containers and images to /var/lib/containerd.
In this blog post, we want to bring attention to ways that you can configure your container runtime to store its content separately from the default partition.
This allows for more flexibility in configuring Kubernetes and provides support for adding a larger disk for the container storage while keeping the default filesystem untouched.
One area that needs more explaining is where/what Kubernetes is writing to disk.
Understanding Kubernetes disk usage
Kubernetes has persistent data and ephemeral data. Th...
![](https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.png)

- Highlights first synced by [[Readwise]] [[Jan 31st, 2024]]
	- CRI-O, by default, writes its containers and images to `/var/lib/containers`, while containerd writes its containers and images to `/var/lib/containerd`. ([View Highlight](https://read.readwise.io/read/01hnenrej9gyrt87vgfjfzzet2))
	- Kubernetes has persistent data and ephemeral data. The base path for the kubelet and local Kubernetes-specific storage is configurable, but it is usually assumed to be `/var/lib/kubelet`. In the Kubernetes docs, this is sometimes referred to as the root or node filesystem. The bulk of this data can be categorized into:
	  
	  •   ephemeral storage
	  •   logs
	  •   and container runtime ([View Highlight](https://read.readwise.io/read/01hnenspqn2230rrs1ze79y1f2))
	- By default, Kubernetes stores the logs of each running container, as files within `/var/log`. These logs are ephemeral and are monitored by the kubelet to make sure that they do not grow too large while the pods are running. ([View Highlight](https://read.readwise.io/read/01hnenvq631901tqwhnzne260w))
	- The container runtime has two different areas of storage for containers and images.
	  
	  •   read-only layer: Images are usually denoted as the read-only layer, as they are not modified when containers are running. The read-only layer can consist of multiple layers that are combined into a single read-only layer. There is a thin layer on top of containers that provides ephemeral storage for containers if the container is writing to the filesystem.
	    
	  •   writeable layer: Depending on your container runtime, local writes might be implemented as a layered write mechanism (for example, `overlayfs` on Linux or CimFS on Windows). This is referred to as the writable layer. Local writes could also use a writeable filesystem that is initialized with a full clone of the container image; this is used for some runtimes based on hypervisor virtualisation. ([View Highlight](https://read.readwise.io/read/01hnenxhq5d1nzf79rxxtnnp40))
- New highlights added [[Jan 31st, 2024]] at 11:54 AM
	- If either nodefs or the imagefs are running out of disk space, then the overall node is considered to have disk pressure. Kubernetes will first reclaim space by deleting unusued containers and images, and then it will resort to evicting pods. On a node that has a nodefs and an imagefs, the kubelet will [garbage collect](https://kubernetes.io/docs/concepts/architecture/garbage-collection/#containers-images) unused container images on imagefs and will remove dead pods and their containers from the nodefs. If there is only a nodefs, then Kubernetes garbage collection includes dead containers, dead pods and unused images.
	  
	  Kubernetes allows more configurations for determining if your disk is full.  
	  The eviction manager within the kubelet has some configuration settings that let you control the relevant thresholds. For filesystems, the relevant measurements are `nodefs.available`, `nodefs.inodesfree`, `imagefs.available`, and `imagefs.inodesfree`. If there is not a dedicated disk for the container runtime then imagefs is ignored.
	  
	  Users can use the existing defaults:
	  
	  •   `memory.available` < 100MiB
	  •   `nodefs.available` < 10%
	  •   `imagefs.available` < 15%
	  •   `nodefs.inodesFree` < 5% (Linux nodes)
	  
	  Kubernetes allows you to set user defined values in `EvictionHard` and `EvictionSoft` in the kubelet configuration file.
	  
	  `EvictionHard`
	  
	  defines limits; once these limits are exceeded, pods will be evicted without any grace period.
	  
	  `EvictionSoft`
	  
	  defines limits; once these limits are exceeded, pods will be evicted with a grace period that can be set per signal.
	  
	  If you specify a value for `EvictionHard`, it will replace the defaults.  
	  This means it is important to set all signals in your configuration. ([View Highlight](https://read.readwise.io/read/01hner9asdz64xehma5e2dq2qf))
	- One common misconfiguration administrators or users can hit is mounting a new filesystem to `/var/lib/containers/storage` or `/var/lib/containerd`. Kubernetes will detect a separate filesystem, so you want to make sure to check that `imagefs.inodesfree` and `imagefs.available` match your needs if you've done this. ([View Highlight](https://read.readwise.io/read/01hnersrext51pa4hjfdad0j2v))
	- Another area of confusion is that ephemeral storage reporting does not change if you define an image filesystem for your node. The image filesystem (`imagefs`) is used to store container image layers; if a container writes to its own root filesystem, that local write doesn't count towards the size of the container image. The place where the container runtime stores those local modifications is runtime-defined, but is often the image filesystem. If a container in a pod is writing to a filesystem-backed `emptyDir` volume, then this uses space from the `nodefs` filesystem. The kubelet always reports ephemeral storage capacity and allocations based on the filesystem represented by `nodefs`; this can be confusing when ephemeral writes are actually going to the image filesystem. ([View Highlight](https://read.readwise.io/read/01hnervyh9rknv74bt5kbhbk31))
	- To fix the ephemeral storage reporting limitations and provide more configuration options to the container runtime, SIG Node are working on [KEP-4191](http://kep.k8s.io/4191). In KEP-4191, Kubernetes will detect if the writeable layer is separated from the read-only layer (images). This would allow us to have all ephemeral storage, including the writeable layer, on the same disk as well as allowing for a separate disk for images. ([View Highlight](https://read.readwise.io/read/01hnes11zk2a74a6va8njykqk3))