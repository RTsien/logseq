title:: Understanding Kubernetes Limits and Requests – Sysdig (highlights)
author:: [[sysdig.com]]
full-title:: "Understanding Kubernetes Limits and Requests – Sysdig"
category:: #articles
url:: https://sysdig.com/blog/kubernetes-limits-requests/
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/Kubernetes-Limits-and-Request-01.png)

- Highlights first synced by [[Readwise]] [[Apr 11th, 2024]]
	- Best practices
	  
	  In very few cases should you be using limits to control your resources usage in Kubernetes. This is because if you want to avoid starvation (ensure that every important process gets its share), you should be using requests in the first place. 
	  
	  By setting up limits, you are only preventing a process from retrieving additional resources in exceptional cases, causing an OOM kill in the event of memory, and Throttling in the event of CPU (process will need to wait until the CPU can be used again).
	  
	  For more information, check the [article about OOM and Throttling](https://sysdig.com/blog/troubleshoot-kubernetes-oom/).
	  
	  If you’re setting a request value equal to the limit in all containers of a Pod, that Pod will get the Guaranteed Quality of Service. 
	  
	  Note as well, that Pods that have a resource usage higher than the requests are more likely to be evicted, so setting up very low requests cause more harm than good.For more information, check the article about [Pod eviction and Quality of Service](https://sysdig.com/blog/kubernetes-pod-evicted/). ([View Highlight](https://read.readwise.io/read/01hv40w5jk9t1y4xmt24600zzv))