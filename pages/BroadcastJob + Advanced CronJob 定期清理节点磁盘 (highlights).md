title:: BroadcastJob + Advanced CronJob 定期清理节点磁盘 (highlights)
author:: [[openkruise.io]]
full-title:: "BroadcastJob + Advanced CronJob 定期清理节点磁盘"
category:: #articles
url:: https://openkruise.io/zh/docs/best-practices/acronjob+broadcastjob
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Jan 4th, 2024]]
	- apiVersion: apps.kruise.io/v1alpha1  
	  kind: AdvancedCronJob  
	  metadata:  
	  name: acj-test  
	  spec:  
	  schedule: "*/5 * * * *"  
	  startingDeadlineSeconds: 60  
	  template:  
	  broadcastJobTemplate:  
	  spec:  
	  template:  
	  spec:  
	  containers:  
	  \- name: node-cleaner  
	  image: minchou/cleaner:v1  
	  imagePullPolicy: IfNotPresent  
	  env:
- New highlights added [[Jan 5th, 2024]] at 3:13 PM
	- job.yaml
	  
	    apiVersion: apps.kruise.io/v1alpha1kind: AdvancedCronJobmetadata:name: acj-testspec:schedule: "*/5 * * * *"startingDeadlineSeconds: 60template:broadcastJobTemplate:spec:template:spec:containers:- name: node-cleanerimage: minchou/cleaner:v1imagePullPolicy: IfNotPresentenv:# crictl use this env to find container runtime socket# this value should consistent with the path of mounted # container runtime socket file - name: CONTAINER_RUNTIME_ENDPOINTvalue: unix:///var/run/containerd/containerd.sockvolumeMounts:# mount container runtime socket file to this path- name: containerdmountPath: /var/run/containerdvolumes:- name: containerdhostPath:path: /var/run/containerdrestartPolicy: OnFailurecompletionPolicy:type: AlwaysttlSecondsAfterFinished: 90failurePolicy:type: ContinuerestartLimit: 3
	  
	  因为需要拿到宿主机上的 containerd.socket 才能执行 `crictl rmi` 之类的镜像清理命令，因此此处需要以 hostPath 的方式将该 `containerd.sock` 文件进行挂载。如果你的宿主机上使用的是其他类型的容器运行时，也需要将其以这种方式挂载到 Pod。 ([View Highlight](https://read.readwise.io/read/01hkc66zhbrwsxd6ecsj2kp0g7))