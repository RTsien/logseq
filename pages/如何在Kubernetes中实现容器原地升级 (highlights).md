title:: 如何在Kubernetes中实现容器原地升级 (highlights)
author:: [[O]]
full-title:: "如何在Kubernetes中实现容器原地升级"
category:: #articles
url:: https://cloud.tencent.com/developer/article/1413743
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/cloud.png)

- Highlights first synced by [[Readwise]] [[Jan 5th, 2024]]
	- ![](https://ask.qcloudimg.com/http-save/1642192/zsnct0daiu.jpeg) ([View Highlight](https://read.readwise.io/read/01hkc5f3rfppgqp91mjy5x0z5j))
	- 使用StatefulSet部署一个Demo，然后修改某个Pod的Spec中nginx容器的镜像版本，通过kubelet日志可以发现的确如此。 ([View Highlight](https://read.readwise.io/read/01hkc5dfmrf6n0snsmdm27xqxs))
		- 💡: 注意这里是直接修改pod的spec，而不是statefulset的template spec