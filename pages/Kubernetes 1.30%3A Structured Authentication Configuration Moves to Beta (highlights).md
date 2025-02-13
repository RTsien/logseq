title:: Kubernetes 1.30: Structured Authentication Configuration Moves to Beta (highlights)
author:: [[Kubernetes – Production-Grade Container Orchestration]]
full-title:: "Kubernetes 1.30: Structured Authentication Configuration Moves to Beta"
category:: #articles
url:: https://kubernetes.io/blog/2024/04/25/structured-authentication-moves-to-beta/
summary:: With Kubernetes 1.30, we (SIG Auth) are moving Structured Authentication Configuration to beta.
Today's article is about authentication: finding out who's performing a task, and checking
that they are who they say they are. Check back in tomorrow to find about what's new in
Kubernetes v1.30 around authorization (deciding what someone can and can't access).
Motivation
Kubernetes has had a long-standing need for a more flexible and extensible
authentication system. The current system, while powerful, has some limitations
that make it difficult to use in certain scenarios. For example, it is not
possible to use multiple authenticators of the same type (e.g., multiple JWT
authenticators) or to change the configuration without restarting the API server. The
Structured Authentication Configuration feature is the first step towards
addressing these limitations and providing a more flexible and extensible way
to configure authentication in Kubernetes.
What is structured authentication configuration?
Kubernetes v1.30 bui...
![](https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.png)

- Highlights first synced by [[Readwise]] [[Apr 26th, 2024]]
	- Today's article is about *authentication*: finding out who's performing a task, and checking that they are who they say they are. Check back in tomorrow to find about what's new in Kubernetes v1.30 around *authorization* ([View Highlight](https://read.readwise.io/read/01hwb1vqpk369kp39714bhgfnp))
		- 💡: 今天的文章是关于*认证*：找出谁正在执行任务，并检查他们是否是他们所说的那个人。明天再来看看 Kubernetes v1.30 中关于*授权*方面的新内容。 #英语语料
	- **Dynamic configuration**: You can change the configuration without restarting the API server. This allows you to add, remove, or modify authenticators without disrupting the API server. ([View Highlight](https://read.readwise.io/read/01hwb2682qw7zdnrb7e3vghsnw))
		- 💡: #TODO 可以研究一下原理