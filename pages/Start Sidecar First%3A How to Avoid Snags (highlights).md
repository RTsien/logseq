title:: Start Sidecar First: How to Avoid Snags (highlights)
author:: [[Kubernetes – Production-Grade Container Orchestration]]
full-title:: "Start Sidecar First: How to Avoid Snags"
category:: #articles
url:: https://kubernetes.io/blog/2025/06/03/start-sidecar-first/
summary:: From the Kubernetes Multicontainer Pods: An Overview blog post you know what their job is, what are the main architectural patterns, and how they are implemented in Kubernetes. The main thing I’ll cover in this article is how to ensure that your sidecar containers start before the main app. It’s more complicated than you might think!
A gentle refresher
I'd just like to remind readers that the v1.29.0 release of Kubernetes added native support for
sidecar containers, which can now be defined within the .spec.initContainers field,
but with restartPolicy: Always. You can see that illustrated in the following example Pod manifest snippet:
initContainers:
 \- name: logshipper
 image: alpine:latest
 restartPolicy: Always # this is what makes it a sidecar container
 command: ['sh', '-c', 'tail -F /opt/logs.txt']
 volumeMounts:
 \- name: data
 mountPath: /opt
What are the specifics of defining sidecars with a .spec.initContainers block, rather than as a legacy multi-container pod with multiple .spec.containers?
Well, all .spec.initContainers are always launched before the main application. If you define Kubernetes-native sidecars, those are terminated after the main application. Furthermore, when used with Jobs, a sidecar container should still be alive and could potentially even restart after the owning Job is complete; Kubernetes-native sidecar containers do not block pod completion.
To learn more, you can also read the official Pod sidecar containers tutorial.
The problem
Now you know that defining a sidecar with this native approach will always start it before the main application. From the kubelet source code, it's visible that this often means being started almost in parallel, and this is not always what an engineer wants to achieve. What I'm really interested in is whether I can delay the start of the main application until the sidecar is not just started, but fully running and ready to serve.
It might be a bit tricky because the problem with sidecars is there’s no ...
![](https://kubernetes.io/icons/icon-128x128.png)

- Highlights first synced by [[Readwise]] [[Jun 7th, 2025]]
	- What are the specifics of defining sidecars with a `.spec.initContainers` block, rather than as a legacy multi-container pod with multiple `.spec.containers`? Well, all `.spec.initContainers` are always launched **before** the main application. If you define Kubernetes-native sidecars, those are terminated **after** the main application. Furthermore, when used with [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/), a sidecar container should still be alive and could potentially even restart after the owning Job is complete; Kubernetes-native sidecar containers do not block pod completion. ([View Highlight](https://read.readwise.io/read/01jx4pg2cfr1d2ab3xg1akbx4c))
	- Now you know that defining a sidecar with this native approach will always start it before the main application. From the [kubelet source code](https://github.com/kubernetes/kubernetes/blob/537a602195efdc04cdf2cb0368792afad082d9fd/pkg/kubelet/kuberuntime/kuberuntime_manager.go#L827-L830), it's visible that this often means being started almost in parallel, and this is not always what an engineer wants to achieve. What I'm really interested in is whether I can delay the start of the main application until the sidecar is not just started, but fully running and ready to serve. ([View Highlight](https://read.readwise.io/read/01jx4pm8p4zceteq8e5cczj9w4))
	- To ensure that the sidecar is ready before the main app container starts, I can define a `startupProbe`. It will delay the start of the main container until the command is successfully executed (returns `0` exit status). If you’re wondering why I’ve added it to my `initContainer`, let’s analyse what happens If I’d added it to myapp container. I wouldn’t have guaranteed the probe would run before the main application code - and this one, can potentially error out without the sidecar being up and running. ([View Highlight](https://read.readwise.io/read/01jx4pw5zyd962c8mw16pcrys2))
	- Fun fact: using the `postStart` lifecycle hook block will also do the job, but I’d have to write my own mini-shell script, which is even less efficient. ([View Highlight](https://read.readwise.io/read/01jx4pyaagn8jfzgkgkdtmwbe7))
	- An interesting exercise would be to check the sidecar container behavior with a [liveness probe](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/). A liveness probe behaves and is configured similarly to a readiness probe - only with the difference that it doesn’t affect the readiness of the container but restarts it in case the probe fails. ([View Highlight](https://read.readwise.io/read/01jx4pz0qxvfcdj5a3bxg5gh5g))
	- I’ll summarize the startup behavior in the table below:
	  
	  Probe/Hook
	  
	  Sidecar starts before the main app?
	  
	  Main app waits for the sidecar to be ready?
	  
	  What if the check doesn’t pass?
	  
	  `readinessProbe`
	  
	  **Yes**, but it’s almost in parallel (effectively **no**)
	  
	  **No**
	  
	  Sidecar is not ready; main app continues running
	  
	  `livenessProbe`
	  
	  Yes, but it’s almost in parallel (effectively **no**)
	  
	  **No**
	  
	  Sidecar is restarted, main app continues running
	  
	  `startupProbe`
	  
	  **Yes**
	  
	  **Yes**
	  
	  Main app is not started
	  
	  postStart
	  
	  **Yes**, main app container starts after `postStart` completes
	  
	  **Yes**, but you have to provide custom logic for that
	  
	  Main app is not started ([View Highlight](https://read.readwise.io/read/01jx4q0964r8j877z0nk9ggaab))
	- To summarize: with sidecars often being a dependency of the main application, you may want to delay the start of the latter until the sidecar is healthy. The ideal pattern is to start both containers simultaneously and have the app container logic delay at all levels, but it’s not always possible. If that's what you need, you have to use the right kind of customization to the Pod definition. Thankfully, it’s nice and quick, and you have the recipe ready above. ([View Highlight](https://read.readwise.io/read/01jx4q0svha3znd460an0r9ffv))