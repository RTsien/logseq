title:: Sidecar 的注入与透明流量劫持 (highlights)
author:: [[宋净超（Jimmy Song）]]
full-title:: "Sidecar 的注入与透明流量劫持"
category:: #articles
url:: https://jimmysong.io/kubernetes-handbook/usecases/understand-sidecar-injection-and-traffic-hijack-in-istio-service-mesh.html
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Dec 5th, 2023]]
	- Init 容器使用 Linux Namespace，所以相对应用程序容器来说具有不同的文件系统视图。因此，它们能够具有访问 Secret 的权限，而应用程序容器则不能。
	  
	  在 Pod 启动过程中，Init 容器会按顺序在网络和数据卷初始化之后启动。每个容器必须在下一个容器启动之前成功退出。如果由于运行时或失败退出，将导致容器启动失败，它会根据 Pod 的 `restartPolicy` 指定的策略进行重试。然而，如果 Pod 的 `restartPolicy` 设置为 Always，Init 容器失败时会使用 `RestartPolicy` 策略。 ([View Highlight](https://read.readwise.io/read/01hgwf45qd854cv4hm55abkyhq))
	- 在所有的 Init 容器没有成功之前，Pod 将不会变成 `Ready` 状态。Init 容器的端口将不会在 Service中进行聚集。 正在初始化中的 Pod 处于 `Pending` 状态，但应该会将 `Initializing` 状态设置为 true。Init 容器运行完成以后就会自动终止。 ([View Highlight](https://read.readwise.io/read/01hgwf4t0pp62fxp92sr7kprc8))
	- 参考 `istio-init` 容器的启动参数，完整的启动命令如下：
	  
	    $ /usr/local/bin/istio-iptables -p 15001 -z 15006 -u 1337 -m REDIRECT -i '*' -x "" -b * -d "15090,15201,15020"
	    
	  
	  该容器存在的意义就是让 Envoy 代理可以拦截所有的进出 Pod 的流量，即将入站流量重定向到 [Sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar)，再拦截应用容器的出站流量经过 [Sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 处理后再出站。 ([View Highlight](https://read.readwise.io/read/01hgwf764e70c828r7g94jpc6w))