title:: 为什么MCP传输机制要从SSE转到Streamable HTTP (highlights)
author:: [[俞乾]]
full-title:: "为什么MCP传输机制要从SSE转到Streamable HTTP"
category:: #articles
url:: https://mp.weixin.qq.com/s/U1z_eHHZ11xiHvB_1Rc5cg
summary:: MCP 已将传输从 SSE 换为 Streamable HTTP，以应对 AI 场景对实时性和大数据量的需求。  
Streamable HTTP 利用 HTTP 分块传输，支持可恢复流、无需长连接并能更高效地传输数据。  
它与现有 HTTP 基础设施兼容，提升可伸缩性和鲁棒性，简化端点管理。
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/fY5ccxoFiaKG9GfialBpNBbqInVttalPxVGOssqn4cNDZ2kXTf3XgsRn3j4BeBicVWpXR4U5oT8ic5yalZGWwmpPJg/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Aug 19th, 2025]]
	- 在实时通信领域，WebSocket 常常被认为是全双工通信的理想选择。然而，对于 MCP 这类主要以服务器到客户端流式传输为主，偶尔需要客户端发送请求的场景，WebSocket 并非总是最佳方案。Streamable HTTP 在与 WebSocket 的权衡中展现出独特的优势：
	  
	  •   • **避免不必要的开销**：对于简单的 RPC 调用或数据流，WebSocket 的全双工特性可能引入不必要的协议开销和复杂性。Streamable HTTP 在保持流式传输能力的同时，更加轻量。
	  •   • **更好的 HTTP 兼容性**：WebSocket 的协议升级机制有时会与现有的 HTTP 基础设施（如代理、负载均衡器）产生兼容性问题，并且浏览器无法直接在 WebSocket 连接上附加 HTTP 头（如 Authorization）。Streamable HTTP 则完全兼容 HTTP，避免了这些问题。
	  •   • **POST 请求的灵活性**：WebSocket 的升级握手主要基于 GET 请求，这使得基于 POST 的复杂交互流程实现起来较为繁琐。Streamable HTTP 则对 POST 和 GET 请求都提供了良好的支持。 ([View Highlight](https://read.readwise.io/read/01k2z0qfhcatmxdzpqg771dmvf))