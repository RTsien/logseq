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