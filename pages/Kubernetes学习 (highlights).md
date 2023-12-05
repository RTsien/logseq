title:: Kubernetes学习 (highlights)
author:: [[Z.S.K.]]
full-title:: "Kubernetes学习"
category:: #articles
url:: https://izsk.me/2023/02/09/Kubernetes-Out-Of-Memory-1/
![](https://luckerbyhome.files.wordpress.com/2021/11/linux_overcommit_1tb.png?w=1024)

- Highlights first synced by [[Readwise]] [[Nov 30th, 2023]]
	- 另一个重要的方面是**OOM-killer不理解Kubernetes pod的概念**。它甚至不知道什么是容器。**容器是构建在cgroup和namespace之上的构造**，因此oom-killer不`理解`容器，**它只知道当一个cgroup超过其内存限制时，它必须杀死该cgroup中的至少一个任务（或进程）**。这对于Kubernetes来说是一个问题，因为它在最底层处理容器。因此，让容器内的进程消失——尤其是任何不是主容器进程的进程——会让Kubernetes陷入困境：容器刚刚被条掉了进程，但还有可能pid 为1的进程还存在，但有可能它已经无法工作。这个确切的问题出现在这里的这个旧线程中[具有多个进程的容器在OOM时未终止](https://github.com/kubernetes/kubernetes/issues/50632)告诉容器中的子进程被终止，整个pod没有标记为OOM。正如Kubernetes的一位委员所说：**只有当pid为1的程序为OOM-killer杀死时，Containers才会被标记为OOM killed**，有些应用程序可以容忍非init进程的OOM kills，因此我们选择不跟踪非init进程OOM kill事件，这是预期的方式。 ([View Highlight](https://read.readwise.io/read/01hge39d50rx1dskka149823cf))
- New highlights added [[Nov 30th, 2023]] at 2:35 AM
	- :01:55 aks-agentpool-20086390-vmss00003C kernel: [ 1432.394642] oom-kill:constraint=CONSTRAINT_MEMCG,nodemask=(null),cpuset=f90b24151029555d49a49d82159ec90c4fec53ba8515bd51a5633d1ff45d8f53,mems_allowed=0,oom_memcg=/kubepods,task_memcg=/kubepods/besteffort/pod5f3d2447-f535-4b3d-979c-216d4980cc3f/f ([View Highlight](https://read.readwise.io/read/01hgfj9kgrax8n56eqm8jqkpt2))
	- •   设置overcommit后，系统对内存的处理是使用时分配，而不是申明时分配
	    
	  •   即使关闭了overcommit，当系统内存不足时，OOM-killer仍将被调用。它的任务是在内存不足时杀死进程，它并不特别关心overcommit是否打开 ([View Highlight](https://read.readwise.io/read/01hgfjaxz5mxmkqf7g7411cqdw))
	- 在Kubernetes中，只有当pid为1的程序为OOM-killer杀死时，Containers才会被标记为OOM killed, 有些应用程序可以容忍非init进程的OOM kill，因此现在kubernetes并不会跟踪非init进程OOM kill事件，目前认为是预期的现象 ([View Highlight](https://read.readwise.io/read/01hgfjf3r6pvr0ye7bf2r5gwjn))