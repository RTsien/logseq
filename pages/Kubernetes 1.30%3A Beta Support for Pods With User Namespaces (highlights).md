title:: Kubernetes 1.30: Beta Support for Pods With User Namespaces (highlights)
author:: [[Kubernetes – Production-Grade Container Orchestration]]
full-title:: "Kubernetes 1.30: Beta Support for Pods With User Namespaces"
category:: #articles
url:: https://kubernetes.io/blog/2024/04/22/userns-beta/
summary:: Kubernetes 1.30 introduces beta support for user namespaces in pods, isolating user and group identifiers to enhance security. User namespaces help limit the impact of container breakouts by restricting privileges on the host and reducing access to files. This feature is significant for enhancing container security and host isolation in Kubernetes environments.
![](https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.png)

- Highlights first synced by [[Readwise]] [[Apr 22nd, 2024]]
	- One Linux namespace that was left behind is the [user namespace](https://man7.org/linux/man-pages/man7/user_namespaces.7.html). This namespace allows us to isolate the user and group identifiers (UIDs and GIDs) we use inside the container from the ones on the host.
	  
	  This is a powerful abstraction that allows us to run containers as "root": we are root inside the container and can do everything root can inside the pod, but our interactions with the host are limited to what a non-privileged user can do. This is great for limiting the impact of a container breakout.
	  
	  A container breakout is when a process inside a container can break out onto the host using some unpatched vulnerability in the container runtime or the kernel and can access/modify files on the host or other containers. If we run our pods with user namespaces, the privileges the container has over the rest of the host are reduced, and the files outside the container it can access are limited too. ([View Highlight](https://read.readwise.io/read/01hw1whg0m5zss29emwbdeszq3))