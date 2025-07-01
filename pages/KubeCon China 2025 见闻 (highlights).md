title:: KubeCon China 2025 见闻 (highlights)
author:: [[This Cute World]]
full-title:: "KubeCon China 2025 见闻"
category:: #articles
url:: https://thiscute.world/posts/kubecon-china-2025/?utm_source=atom_feed
summary:: 前言今年 1 月底辞职后，在家过了个年，接着上海、张家界、重庆、苏州、南京玩了一圈，4 月中旬才回深圳开始找工作。本来看到 6 月就是 KubeCon China 2025，还不太确定自己到时候会不会有时间去。不过很幸运，最后确定 offer 的公司非常 Tech，leader 在面试的时候就说看到我博客里写了
KubeCon 的经历，公司非常鼓励参加这种技术交流活动，去报个 Talk 也完全可以，公司报销所有费用。
于是我在入职还没满一个月的时候，就直接公费出差去香港 KubeCon China 2025 玩了一圈（

也问过同事们是否有想法，但种种原因最后还是只有我一个人参加了（悲
![](https://thiscute.world/posts/kubecon-china-2025/featured-image.webp)

- Highlights first synced by [[Readwise]] [[Jun 19th, 2025]]
	- 大一统的 LLM 推理解决方案
	  
	  •   [Introducing AIBrix: Cost-Effective and Scalable Kubernetes Control Plane for VLLM - Jiaxin Shan & Liguang Xie, ByteDance](https://kccncchn2025.sched.com/event/1x5im/introducing-aibrix-cost-effective-and-scalable-kubernetes-control-plane-for-vllm-jiaxin-shan-liguang-xie-bytedance?iframe=no)
	  
	  AIBrix 则是一整套在 K8s 上跑 LLM 分布式推理的解决方案，它包含了：
	  
	  •   分布式推理的部署
	  •   LLM 扩缩容
	  •   LLM 请求路由（负载均衡）
	  •   分布式 KV 缓存
	    •   主要是中心化存储这些数据，减少对 HMB 显存的使用，降低显存需求。
	  •   LoRa 的动态加载
	  •   …
	  
	  AIBrix 目前放在了 vllm-project 项目下，stars 也不少，感觉项目还是挺健康的，值得关注。
	  
	  [](https://thiscute.world/posts/kubecon-china-2025/#%e5%88%86%e5%b8%83%e5%bc%8f-llm-%e6%8e%a8%e7%90%86%e7%9a%84%e9%83%a8%e7%bd%b2)分布式 LLM 推理的部署
	  
	  [More Than Model Sharding: LWS & Distributed Inference - Peter Pan & Nicole Li, DaoCloud & Shane Wang, Intel](https://kccncchn2025.sched.com/event/1x5i6/more-than-model-sharding-lws-distributed-inference-peter-pan-nicole-li-daocloud-shane-wang-intel?iframe=no&w=100%25&sidebar=yes&bg=no)
	  
	  全场最有意思的 Talks 之一，大概介绍了分布式推理的架构、优化点，以及 LWS 的优点与用法。
	  
	  简单的说 LWS 是一个专门为 LLM 分布式推理的部署而设计的 CRD, 主要是支持了 LLM 任务的分组调度。
	  
	  NOTE: 看 issue AIBrix 还有跟 LWS 结合使用的可能性（甚至可能被官方支持）:[https://github.com/vllm-project/aibrix/issues/843#issuecomment-2728305020](https://github.com/vllm-project/aibrix/issues/843#issuecomment-2728305020) ([View Highlight](https://read.readwise.io/read/01jy234tm0sxw5bgqd09mnm93h))
		- 💡: 学习一下 k8s 自动化部署的新思路
	- 大一统的 LLM 推理解决方案
	  
	  •   [Introducing AIBrix: Cost-Effective and Scalable Kubernetes Control Plane for VLLM - Jiaxin Shan & Liguang Xie, ByteDance](https://kccncchn2025.sched.com/event/1x5im/introducing-aibrix-cost-effective-and-scalable-kubernetes-control-plane-for-vllm-jiaxin-shan-liguang-xie-bytedance?iframe=no)
	  
	  AIBrix 则是一整套在 K8s 上跑 LLM 分布式推理的解决方案，它包含了：
	  
	  •   分布式推理的部署
	  •   LLM 扩缩容
	  •   LLM 请求路由（负载均衡）
	  •   分布式 KV 缓存
	    •   主要是中心化存储这些数据，减少对 HMB 显存的使用，降低显存需求。
	  •   LoRa 的动态加载
	  •   …
	  
	  AIBrix 目前放在了 vllm-project 项目下，stars 也不少，感觉项目还是挺健康的，值得关注。
	  
	  [](https://thiscute.world/posts/kubecon-china-2025/#%e5%88%86%e5%b8%83%e5%bc%8f-llm-%e6%8e%a8%e7%90%86%e7%9a%84%e9%83%a8%e7%bd%b2)分布式 LLM 推理的部署
	  
	  [More Than Model Sharding: LWS & Distributed Inference - Peter Pan & Nicole Li, DaoCloud & Shane Wang, Intel](https://kccncchn2025.sched.com/event/1x5i6/more-than-model-sharding-lws-distributed-inference-peter-pan-nicole-li-daocloud-shane-wang-intel?iframe=no&w=100%25&sidebar=yes&bg=no) ([View Highlight](https://read.readwise.io/read/01jy234ep50gppqddn9a9xy4g7))
	- [KubeCon EU 2025 - Keynote: LLM-Aware Load Balancing in Kubernetes: A New Era of Efficiency - Clayton Coleman, Distinguished Engineer, Google & Jiaxin Shan, Software Engineer, Bytedance](https://www.youtube.com/watch?v=BBqDpqATcI0&list=PLj6h78yzYM2MP0QhYFK8HOb8UqgbIkLMc&index=26)
	  
	  •   很有意思，LLM 的请求跟传统的 API 请求区别非常大，主要点在于：
	    •   input 长度区别就非常大，有的请求 input 很简单，相对就很轻量，而有的可能直接丢一份 PDF 或者别的超长文本输入。输出也同样如此，如果用户明确要求深度推理，可能会导致大量性能消耗。
	    •   不同机器可能会使用不同的 GPU 类型，而这些 GPU 的性能各异。
	    •   在一个支持多模型的平台上，不同模型的高低峰期也存在比较明显的区别。
	  •   上面这些特征导致传统的负载均衡策略完全失效。 ([View Highlight](https://read.readwise.io/read/01jy236y511bbce79001c8asdq))
	- [AI Model Distribution Challenges and Best Practices](https://kccncchn2025.sched.com/event/1x5hl/ai-model-distribution-challenges-and-best-practices-wenbo-qi-xiaoya-xia-peng-tao-ant-group-wenpeng-li-alibaba-cloud-han-jiang-kuaishou?iframe=no&w=100%25&sidebar=yes&bg=no)
	  
	  •   几位开发者聊怎么在集群里分发数百 GB 大小的 LLM 模型。
	  •   业界目前的手段：dragonfly, juicefs, oci model spec + oci volume (k8s 1.33+) ([View Highlight](https://read.readwise.io/read/01jy237kyj6eb6cbswvn69s6e4))
		- 💡: 大容量镜像/数据分发
	- [KubeCon EU 2025 - From Logs To Insights: Real-time Conversational Troubleshooting for Kubernetes With GenAI - Tiago Reichert & Lucas Duarte, AWS](https://www.youtube.com/watch?v=7yhBBzVmPks)
	  
	  •   开场的 OnCall 小品就很真实… 不过 pod pending 1 分钟就电话告警有点夸张了…
	  •   演完小品才开始讲正式内容，大体上就是把日志用 embed 模型编码后存在 OpenSearch 里做 RAG，还给了 ChatBot k8s readonly 的权限（ban 掉了 secrets access），然后通过 Deepseek/Claude 问答来解决问题。
	  •   代码: [https://github.com/aws-samples/sample-eks-troubleshooting-rag-chatbot](https://github.com/aws-samples/sample-eks-troubleshooting-rag-chatbot) ([View Highlight](https://read.readwise.io/read/01jy23atrjj5f1x4phemv2teqz))
		- 💡: 嵌入流程可以学习一下，这么大的数据量都存下来了？？？
	- [Revolutionizing Sidecarless Service Mesh With eBPF - Zhonghu Xu & Muyang Tian, Huawei](https://kccncchn2025.sched.com/event/1x5iI/revolutionizing-sidecarless-service-mesh-with-ebpf-zhonghu-xu-muyang-tian-huawei)
	  
	  •   主要就讲 Huawei 自己搞的 Kmesh，有比较详细的讲底层的实现架构（其实跟去年 KubeCon 听过的内容几乎一样）。
	  •   简单讲就是 Ambient Mode 通过 istio-cni（底层是 iptables）将流量拦截到用户态的 ztunnel 进行 L4 流量处理，而 Kmesh 使用 eBPF 在内核层实现了这些 L4 的功能。另外还简单介绍了 Cilium Service Mesh，是一个 Per-Node 的 Proxy，主要缺点是必须用 Cilium 网络插件，以及它的 CRD 过于原始，使用复杂。
	  •   Kmesh 也尝试用 eBPF 实现了 HTTP 协议的解析，但是这需要对内核打补丁，代价比较高。 ([View Highlight](https://read.readwise.io/read/01jy23dncvpfzapmf7wf5jrtjw))
	- [KubeCon EU 2025 - Navigating the Maze of Multi-Cluster Istio: Lessons Learned at Scale - Pamela Hernandez, BlackRock](https://www.youtube.com/watch?v=WpEkfVGWmd8)
	  
	  •   Istio 多集群在挺多大公司有应用，之前面试就被问到过，可以玩玩看 ([View Highlight](https://read.readwise.io/read/01jy23kp491v8v4ehxvx0j8wqn))
	- [KubeCon EU 2025 - A Service Mesh Benchmark You Can Trust - Denis Jannot, solo.io](https://www.youtube.com/watch?v=oi4TpxuIYXk)
	  
	  •   做一个好的 Benchmark 对比还挺费时间费精力的，还是直接看人家给的结果最方便（ ([View Highlight](https://read.readwise.io/read/01jy23mbc7cc6zc9v0jwvtvhf9))
	- [KubeCon EU 2025 - Autonomous Al Agents for Cloud Cost Analysis - Ilya Lyamkin, Spotify](https://www.youtube.com/watch?v=sTbJ1-x3_yc&list=PLj6h78yzYM2MP0QhYFK8HOb8UqgbIkLMc&index=345)
	  
	  •   实现一个会自动做 Plan，编写 SQL 与 Python 进行云生成分析的 Multi-Agent 系统，很有参考价值。 ([View Highlight](https://read.readwise.io/read/01jy23na1h9vqv4gze13rnncm6))
		- 💡: 学习多 agent 系统的开发模式