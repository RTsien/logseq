title:: 如何准确识别K8S中的PodOOMKilled并发出报警 (highlights)
author:: [[ipcpu.com]]
full-title:: "如何准确识别K8S中的PodOOMKilled并发出报警"
category:: #articles
url:: https://www.ipcpu.com/2023/06/k8s-pod-oom-killed/
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Nov 30th, 2023]]
	- *只有当pid为1的程序为OOM-killer杀死时，Containers才会被标记为OOM killed* ([View Highlight](https://read.readwise.io/read/01hge2psc3q1y89jbtpbq2qnf4))
	- 对于Node节点系统 /dev/kmsg 文件的监控，Kubenertes官方是有工具的，叫NPD(node-problem-detector)，链接如下
	  
	  [https://github.com/kubernetes/node-problem-detector](https://github.com/kubernetes/node-problem-detector)
	  
	  这个工具会监控/dev/kmsg 文件的报错信息、Docker进程是否存在等信息，然后将发现的问题以Event形式上报给 K8S的apiserver，这样使用 kubectl get event 就可以看到对应的信息了。
	  
	  但是呢，官方的这个npd有很大的缺陷，那就是拿不到被Kill的容器名称，这个很重要，如果没有容器名称，只有pid的话，根本无法排查，并且POD被Kill了，根本找不对对应关系。
	  
	  好在阿里云在它自身的Kubernetes服务中提供了增强版npd，可以实现这个获取容器名称的功能。 ([View Highlight](https://read.readwise.io/read/01hge2vhbpxgvnm9tnjvnm5nww))
	- 但是需要注意的地方就是刚刚说的，UUID的获取，当前阿里的版本没有考虑到使用中横线连接UUID的情况，导致5.4内核版本拿不到UUID，不仅如此，阿里的版本在5.4也存在正则匹配的问题，如果你不幸使用了5.4版本，都需要自行修改代码适配。
	  
	  我把我的NPD部署的YAML放在了github，大家可以参考下。
	  
	  [https://github.com/ipcpu/npd-yaml/blob/main/npd.yaml](https://github.com/ipcpu/npd-yaml/blob/main/npd.yaml) ([View Highlight](https://read.readwise.io/read/01hge2zt55z99nc8s2gx6vg5r0))
	- 有了NPD模块，我们使用K8S的eventrouter功能就可以发到统一汇总的地址了，这里我们是输出到kafka，然后扔进了阿里的SLS，使用了阿里云SLS报警系统。 ([View Highlight](https://read.readwise.io/read/01hge2zejf8eebttaej19yreaw))
- New highlights added [[Dec 5th, 2023]] at 8:35 PM
	- 在K8S中可以使用如下命令查看POD的UUID信息
		- 💡: 可以加到环境变量，方便在容器内获取 
		       \- name: POD_ID
		          valueFrom:
		            fieldRef:
		              apiVersion: v1
		              fieldPath: metadata.uid