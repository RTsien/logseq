title:: Kubernetes 1.30: Read-Only Volume Mounts Can Be Finally Literally Read-Only (highlights)
author:: [[Kubernetes – Production-Grade Container Orchestration]]
full-title:: "Kubernetes 1.30: Read-Only Volume Mounts Can Be Finally Literally Read-Only"
category:: #articles
url:: https://kubernetes.io/blog/2024/04/23/recursive-read-only-mounts/
summary:: Kubernetes 1.30 introduces fully read-only volume mounts, including support for recursive read-only mounts. This feature enhances security by ensuring that all sub-mounts are read-only within containers. Users need to enable the RecursiveReadOnlyMounts feature gate and use compatible versions of components for this functionality.
![](https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.png)

- Highlights first synced by [[Readwise]] [[Apr 24th, 2024]]
	- Kubernetes 1.30 added a new mount option `recursiveReadOnly` so as to make submounts recursively read-only.
	  
	  The option can be enabled as follows:
	  
	  This is implemented by applying the `MOUNT_ATTR_RDONLY` attribute with the `AT_RECURSIVE` flag using [`mount_setattr(2)`](https://man7.org/linux/man-pages/man2/mount_setattr.2.html) added in Linux kernel v5.12.
	  
	  For backwards compatibility, the `recursiveReadOnly` field is not a replacement for `readOnly`, but is used *in conjunction* with it. To get a properly recursive read-only mount, you must set both fields. ([View Highlight](https://read.readwise.io/read/01hw61f0qhcx35hbx2k9e0ph2p))