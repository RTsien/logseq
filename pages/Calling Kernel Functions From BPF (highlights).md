title:: Calling Kernel Functions From BPF (highlights)
author:: [[Jonathan Corbet]]
full-title:: "Calling Kernel Functions From BPF"
category:: #articles
url:: https://lwn.net/Articles/856005/
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Jan 23rd, 2024]]
	- The immediate driver for this functionality is the implementation of TCP congestion-control algorithms in BPF, a capability that was [added to the 5.6 kernel release](https://lwn.net/Articles/811631/) by Martin KaFai Lau. Actual congestion-control implementations in BPF turned out to reimplement a number of functions that already exist in the kernel, which seems less than fully optimal; it would be better to just use the existing functions in the kernel if possible. The new function-calling mechanism — also implemented by Lau — makes that possible. ([View Highlight](https://read.readwise.io/read/01hmqkkyf88rrvc5cx8cxxsqsb)) #eBPF
		- 💡: eBPF支持调用内核函数的由来
		- 💡：it seems less than fully optimal #card #英语语料
		  id:: 65afc450-1c9e-4c4f-8b81-c52fde040afc
			- 这似乎不是最佳的
	- So far, congestion-control programs are the only program type to make use of this feature, but it is not hard to imagine that others will come in the future. There are a number of interesting questions that are raised by this capability and how it might be used going forward. ([View Highlight](https://read.readwise.io/read/01hmqkvazeknns7c7kmazr266z))