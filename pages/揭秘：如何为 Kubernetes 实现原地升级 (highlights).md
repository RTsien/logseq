title:: 揭秘：如何为 Kubernetes 实现原地升级 (highlights)
author:: [[aliyun.com]]
full-title:: "揭秘：如何为 Kubernetes 实现原地升级"
category:: #articles
url:: https://developer.aliyun.com/article/765421
![](https://img.alicdn.com/tfs/TB1LCE1aQ5E3KVjSZFCXXbuzXXa-200-200.png)

- Highlights first synced by [[Readwise]] [[Jan 5th, 2024]]
	- 简单来说，对于一个已经创建出来的 Pod，在 Pod Spec 中只允许修改 containers/initContainers 中的 image 字段，以及 activeDeadlineSeconds 字段。对 Pod Spec 中所有其他字段的更新，都会被 kube-apiserver 拒绝。 ([View Highlight](https://read.readwise.io/read/01hkc0zsesf0mkcgmfhk5npcmk))
	- ReadinessGate 控制 Pod 是否 Ready
	  
	  在 Kubernetes 1.12 版本之前，一个 Pod 是否处于 Ready 状态只是由 kubelet 根据容器状态来判定：如果 Pod 中容器全部 ready，那么 Pod 就处于 Ready 状态。
	  
	  但事实上，很多时候上层 operator 或用户都需要能控制 Pod 是否 Ready 的能力。因此，Kubernetes 1.12 版本之后提供了一个 readinessGates 功能来满足这个场景。如下：
	  
	    apiVersion: v1
	    kind: Pod
	    spec:
	      readinessGates:
	      \- conditionType: MyDemo
	    status:
	      conditions:
	- 1. 单个 Pod 如何原地升级？
	  
	  由“背景 1”可知，其实我们对一个存量 Pod 的 spec.containers[x] 中字段做修改，kubelet 会感知到这个 container 的 hash 发生了变化，随即就会停掉对应的旧容器，并用新的 container 来拉镜像、创建和启动新容器。
	  
	  由“背景 2”可知，当前我们对一个存量 Pod 的 spec.containers[x] 中的修改，仅限于 image 字段。
	  
	  因此，得出第一个实现原理：**对于一个现有的 Pod 对象，我们能且只能修改其中的 spec.containers[x].image 字段，来触发 Pod 中对应容器升级到一个新的 image。
	  
	  2. 如何判断 Pod 原地升级成功？
	  
	  接下来的问题是，当我们修改了 Pod 中的 spec.containers[x].image 字段后，如何判断 kubelet 已经将容器重建成功了呢？
	  
	  由“背景 3”可知，比较 spec 和 status 中的 image 字段是不靠谱的，因为很有可能 status 中上报的是 Node 上存在的另一个镜像名（相同 imageID）。
	  
	  因此，得出第二个实现原理：**判断 Pod 原地升级是否成功，相对来说比较靠谱的办法，是在原地升级前先将 status.containerStatuses[x].imageID 记录下来。在更新了 spec 镜像之后，如果观察到 Pod 的 status.containerStatuses[x].imageID 变化了，我们就认为原地升级已经重建了容器。**
	  
	  但这样一来，我们对原地升级的 image 也有了一个要求：**不能用 image 名字（tag）不同、但实际对应同一个 imageID 的镜像来做原地升级，否则可能一直都被判断为没有升级成功（因为 status 中 imageID 不会变化）。**
	  
	  当然，后续我们还可以继续优化。OpenKruise 即将开源镜像预热的能力，会通过 DaemonSet 在每个 Node 上部署一个 NodeImage Pod。通过 NodeImage 上报我们可以得知 pod spec 中的 image 所对应的 imageID，然后和 pod status 中的 imageID 比较即可准确判断原地升级是否成功。
	  
	  3. 如何确保原地升级过程中流量无损？
	  
	  在 Kubernetes 中，一个 Pod 是否 Ready 就代表了它是否可以提供服务。因此，像 Service 这类的流量入口都会通过判断 Pod Ready 来选择是否能将这个 Pod 加入 endpoints 端点中。
	  
	  由“背景 4”可知，从 Kubernetes 1.12+ 之后，operator/controller 这些组件也可以通过设置 readinessGates 和更新 pod.status.conditions 中的自定义 type 状态，来控制 Pod 是否可用。
	  
	  因此，得出第三个实现原理：**可以在 pod.spec.readinessGates 中定义一个叫 InPlaceUpdateReady 的 conditionType。**
	  
	  在原地升级时：
	  
	  1.  **先将 pod.status.conditions 中的 InPlaceUpdateReady condition 设为 "False"，这样就会触发 kubelet 将 Pod 上报为 NotReady，从而使流量组件（如 endpoint controller）将这个 Pod 从服务端点摘除；**
	  2.  **再更新 pod spec 中的 image 触发原地升级。**
	  
	  **原地升级结束后，再将 InPlaceUpdateReady condition 设为 "True"，使 Pod 重新回到 Ready 状态。**
	  
	  另外在原地升级的两个步骤中，第一步将 Pod 改为 NotReady 后，流量组件异步 watch 到变化并摘除端点可能是需要一定时间的。因此我们也提供优雅原地升级的能力，即通过 gracePeriodSeconds 配置在修改 NotReady 状态和真正更新 image 触发原地升级两个步骤之间的静默期时间。 ([View Highlight](https://read.readwise.io/read/01hkc16rr0z9nncjdm7zsq2m7f))