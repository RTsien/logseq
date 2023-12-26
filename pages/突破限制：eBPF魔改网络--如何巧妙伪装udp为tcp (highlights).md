title:: 突破限制：eBPF魔改网络--如何巧妙伪装udp为tcp (highlights)
author:: [[woa.com]]
full-title:: "突破限制：eBPF魔改网络--如何巧妙伪装udp为tcp"
category:: #articles
url:: https://km.woa.com/group/11879/articles/show/559102?from=km_daily_bot
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Dec 25th, 2023]]
	- XDP 是一种高效的数据包处理技术，特别适用于入站方向的快速处理。它在网络堆栈的早期阶段介入，这意味着能够以极高的效率进行数据包过滤和转发，从而显著提高整体性能。然而，它主要限于处理入站流量，并且在某些复杂网络任务上可能受限。
	  
	  TC 提供了处理Ingress和Egress流量的能力，适用于更广泛的场景，包括流量整形和分类。尽管如此，TC 在网络栈中的处理时机相对较晚，这可能在高速网络环境中对性能产生影响，同时也有一些其他不利限制（参考TCP校验和章节）。
	  
	  根据这些优势和限制，我们最终可以敲定如下方案：
	  
	  ●采用TC/eBPF程序处理Egress方向的UDP数据包，将原始的UDP包头（8字节）伪装成TCP头（20字节）。
	  
	  ●采用XDP/eBPF程序处理Ingress方向的TCP数据包，将检查TCP伪装头，并将其恢复成原始UDP包。 ([View Highlight](https://read.readwise.io/read/01hjfyyfeza2hvr8hnj01t20ck))