title:: 使用 Geneve 隧道实现 Istio Ambient Mesh 的流量拦截 (highlights)
author:: [[Jimmy Song]]
full-title:: "使用 Geneve 隧道实现 Istio Ambient Mesh 的流量拦截"
category:: #articles
url:: https://jimmysong.io/blog/traffic-interception-with-geneve-tunnel-with-istio-ambient-mesh/
summary:: The text explains how Geneve tunnels are used for traffic interception in Istio Ambient Mesh, comparing them to VXLAN. It also discusses the advantages of Geneve tunnels, their application in Istio Ambient Mesh, and the option to use eBPF for traffic interception in Istio 1.18.
![](https://jimmysong.io/images/banner/tunnel.jpg)

- Highlights first synced by [[Readwise]] [[Apr 20th, 2024]]
	- Geneve vs VXLAN[](https://jimmysong.io/blog/traffic-interception-with-geneve-tunnel-with-istio-ambient-mesh#geneve-vs-vxlan)
	  
	  VXLAN 和 Geneve 都是网络虚拟化协议，它们之间有很多共同点。虚拟化协议是一种将虚拟网络与物理网络分离的技术，它允许网络管理员在虚拟环境中创建多个虚拟网络，每个虚拟网络都可以拥有自己的 VLAN 标识符、IP 地址和路由。此外，VXLAN 和 Geneve 协议都使用 UDP 封装，这使得它们能够通过现有网络基础设施进行扩展。VXLAN 和 Geneve 协议还具有灵活性，它们可以在不同的网络拓扑结构中使用，并且可以与不同的虚拟化平台兼容。 ([View Highlight](https://read.readwise.io/read/01hvx0vd0aswp9vcdeq3c60twf))
	- VXLAN 与 Geneve 协议的报文结构及其各自的 Header 区别。
	  
	  ![image](https://jimmysong.io/blog/traffic-interception-with-geneve-tunnel-with-istio-ambient-mesh/vxlan-vs-geneve.svg)
	  
	  图 1：VXLAN 与 Geneve 报文格式示意图
	  
	  从图中我们可以看到，VXLAN 与 Geneve 隧道报文的结构类似，其主要区别在于使用不同的 UDP 端口号和协议头 ——VXLAN 使用 4789 端口，Geneve 使用 6081 端口；Geneve 协议头比 VXLAN 更具扩展性。
	  
	  Geneve 隧道协议比 VXLAN 更加可扩展是因为 Geneve 隧道协议中增加了变长选项，它可以包含零或多个 TLV 格式的选项数据。TLV 是指类型 - 长度 - 值（Type-Length-Value）格式，用于传输和解析网络包的元数据信息。在 Geneve 协议中，每个元数据信息都由一个 TLV 格式的字段组成，以便于灵活地添加、删除和修改这些元数据。
	  
	  具体来说，TLV 格式的字段包括：
	  
	  •   Type：8 位的类型字段。
	  •   Length：5 位的选项长度字段，以 4 字节倍数表示，不包括选项头。
	  •   Data：可变长的选项数据字段，可以不存在或者为 4 到 128 字节之间。
	  
	  通过使用 TLV 格式，Geneve 协议可以轻松地扩展和修改元数据信息，同时保持兼容性和灵活性。 ([View Highlight](https://read.readwise.io/read/01hvx0wb590s8pzjn6y6cym7kr))