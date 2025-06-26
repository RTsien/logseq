title:: Using eBPF for traffic redirection in Istio ambient mode (highlights)
author:: [[Iris Ding - Intel]]
full-title:: "Using eBPF for traffic redirection in Istio ambient mode"
category:: #articles
url:: https://istio.io/latest/blog/2023/ambient-ebpf-redirection/
summary:: In Istio's ambient mode, eBPF is now supported for traffic redirection, offering better performance and flexibility compared to iptables. Istio CNI component utilizes eBPF programs attached to traffic hooks for efficient routing between application pods and ztunnel pods. Users can enable eBPF redirection in ambient mode by setting a configuration parameter during Istio installation.
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/istio-social.svg)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- How to enable eBPF redirection in Istio ambient mode[](https://istio.io/latest/blog/2023/ambient-ebpf-redirection#how-to-enable-ebpf-redirection-in-istio-ambient-mode)
	  
	  Follow the instructions in [Getting Started with Ambient Mesh](https://istio.io/latest/blog/2022/get-started-ambient/) to set up your cluster, with a small change: when you install Istio, set the `values.cni.ambient.redirectMode` configuration parameter to `ebpf`.
	  
	    $ istioctl install --set profile=ambient --set values.cni.ambient.redirectMode="ebpf"
	    
	  
	  Check the `istio-cni` logs to confirm eBPF redirection is on:
	  
	    ambient Writing ambient config: {"ztunnelReady":true,"redirectMode":"eBPF"} ([View Highlight](https://read.readwise.io/read/01hv1rvakn76f99qmcnah1e39c))
	- The latency and throughput (QPS) for eBPF redirection are somewhat better than using iptables. The following tests were run in a `kind` cluster with a Fortio client sending requests to a Fortio server, both running in ambient mode (with eBPF debug logging disabled) and on the same Kubernetes worker node.
	  
	    $ fortio load -uniform -t 60s -qps 0 -c <num_connections> http://<fortio-svc-name>:8080
	    
	  
	  [![Max QPS with varying number of connections](https://istio.io/latest/blog/2023/ambient-ebpf-redirection/MaxQPS.png)](https://istio.io/latest/blog/2023/ambient-ebpf-redirection/MaxQPS.png)
	  
	  Max QPS, with varying number of connections
	  
	    $ fortio load -uniform -t 60s -qps 8000 -c <num_connections> http://<fortio-svc-name>:8080
	    
	  
	  [![Latency (ms) for QPS 8000 with varying number of connections](https://istio.io/latest/blog/2023/ambient-ebpf-redirection/P75-Latency-with-8000-qps.png)](https://istio.io/latest/blog/2023/ambient-ebpf-redirection/P75-Latency-with-8000-qps.png)
	  
	  P75 Latency (ms) for QPS 8000 with varying number of connections ([View Highlight](https://read.readwise.io/read/01hv1rxbe2yng7qbkw03e2xvjn))
- New highlights added [[Jun 11th, 2025]] at 2:36 AM
	- Why eBPF
	  
	  Although performance considerations are essential in the implementation of Istio ambient mode redirection, it’s also important to consider ease of programmability, to enable the implementation of versatile and customized requirements. With eBPF, you can leverage additional context in the kernel to bypass complex routing and simply send packets to their final destination.
	  
	  Furthermore, eBPF enables deeper visibility and additional context for packets in the kernel, allowing for more efficient and flexible management of data flow compared with iptables. ([View Highlight](https://read.readwise.io/read/01jxda2wbkfmx8gm5dvtzypaf2))