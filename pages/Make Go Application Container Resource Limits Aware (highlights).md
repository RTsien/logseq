title:: Make Go Application Container Resource Limits Aware (highlights)
author:: [[kupczynski.info]]
full-title:: "Make Go Application Container Resource Limits Aware"
category:: #articles
url:: https://kupczynski.info/posts/go-container-aware/
summary:: Go binaries are not inherently container-aware, specifically in that they do not account for memory and CPU limits
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- `GOMEMLIMIT` gives the Go runtime a target to hit. The new heap won’t exceed `GOMEMLIMIT` as long as the live heap fits in. Of course, it’s not a miracle; if there’s only a little room left above the live heap, the garbage collector will need to run more often, potentially introducing a lot of latency. It’s also a “soft” target; if the live heap needs more memory than `GOMEMLIMIT` it will still ask the OS for it (which will trigger the oom-killer). ([View Highlight](https://read.readwise.io/read/01j243n9nkhj2ysgzf1f69e4bq))
		- 💡: 即使配了GOMEMLIMIT，仍然有可能oom
	- The impact of setting CPU shares and quotas (requests and limits in Kubernetes terminology) is fascinating. I recommend the following articles on the subject: (1) [“The container throttling problem”](https://danluu.com/cgroup-throttling/) by *Dan Luu* discusses the impact of CPU limits on tail latency of servers within Twitter, (2) [“For the Love of God, Stop Using CPU Limits on Kubernetes”](https://home.robusta.dev/blog/stop-using-cpu-limits) by robusta discusses that it may be better to set only requests, but not limits. Specifically for Go, check (3) [“Go, Containers, and the Linux Scheduler”](https://www.riverphillips.dev/blog/go-cfs/) by River Phillips. ([View Highlight](https://read.readwise.io/read/01j242xeaft13gcs347vpwcknv))
		- 💡: The impact of setting CPU shares and quotas (requests and limits in Kubernetes terminology) is fascinating.
	- The impact of setting CPU shares and quotas (requests and limits in Kubernetes terminology) is fascinating. ([View Highlight](https://read.readwise.io/read/01j2445d8d0hen8jmakzw6rkex))
	- This situation is not a problem when we have a batch process or long running computation of some sort. This is problematic in the case of interactive applications, e.g. serving requests. Let’s say our service generates a reply in 10ms. If it happens to receive a request at 49ms into a period and uses all 4 cores, the response will be generated only in the next period. The reply for this request is generated in 60ms instead of 10ms; the *tail latency* increased. ([View Highlight](https://read.readwise.io/read/01j2437tjpz8tpsh8r5w7m7n7e))
	- Note that selecting the value for `GOMAXPROCS` is a bit harder than for `GOMEMLIMIT`, because:
	  
	  •   Sometimes you are OK with the increased tail latency (e.g. in batch processing, or if each request already takes much longer to process than the CFS period).
	  •   You may have your quota set quite high (or not set at all), but due to the system being under load, you are still throttled. In that case, you may want to set it to a lower value than the quota. ([View Highlight](https://read.readwise.io/read/01j244677m1wc6v1gftttxbcbm))
		- 💡: go runtime的CPU限制要比内存限制难配