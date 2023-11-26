title:: Gomaxprocs 与容器的相处之道 (highlights)
author:: [[高策]]
full-title:: "Gomaxprocs 与容器的相处之道"
category:: #articles
url:: http://gaocegege.com/Blog/maxprocs-cpu
tags:: #[[favorite]] 
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Nov 26th, 2023]]
	- Goroutines 是 Golang 最吸引人的特性之一，它是 stackful coroutines 的一种实现。为了支持这一特性，Golang 需要一个运行时，在 Goroutines 和系统线程之间进行调度，这也就是 go-scheduler 的作用。go-scheduler 引入了三个抽象，分别是 Processor，Machine（简称 M） 和 Goroutine（简称 G）。其中 G 就是用户创建的 goroutines，而 M 则是系统线程，是负责真正执行 goroutines 的系统线程。 Processor 是类似于 CPU 核心的概念，其用来控制并发的 M 数量。 ([View Highlight](https://read.readwise.io/read/01hg3ncsrf2y6skz9ytxj5qegj)) #[[golang]] #[[card]]
	- 当 M 需要执行 G 的时候，它需要寻找到一个空闲的 P，只有跟一个 P 绑定后，M 才能被执行。通过这样的方式，go-scheduler 保证了在同一时间内，最多只有 P 个系统线程在真正地执行。P 的数量在默认情况下，会被设定为 CPU 的数量。而 M 虽然需要跟 P 绑定执行，但数量上并不与 P 相等。这是因为 M 会因为系统调用或者其他事情被阻塞，因此随着程序的执行，M 的数量可能增长，而 P 在没有用户干预的情况下，则会保持不变。 ([View Highlight](https://read.readwise.io/read/01hg3nd48zmpxtb2vr36jfn9x2)) #[[golang]] #[[card]]
	- go-scheduler 确定 P 数量的逻辑。在 Linux 上，它会利用系统调用 [sched_getaffinity](https://linux.die.net/man/2/sched_getaffinity) 来获得系统的 CPU 核数 ([View Highlight](https://read.readwise.io/read/01hg3neyysa87zekdh4080s3h0)) #[[golang]] #[[card]]
	- [CFS Bandwith Control](https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt) 原本是为了解决 CPU Share 不能做 hard limit 的问题的，但它同样造成了新的问题，系统调用 `sched_getaffinity` 并不感知它对进程的限制。这也使得运行在 Kubernetes 中的 Go 程序的运行时始终会认为自己可以使用宿主机上的所有 CPU，进而创建了相同数量的 P。而当其 `GOMAXPROCS` 被手动地设置为限制后的值后，其在 CPU 密集的任务上的表现得到了很大程度的提高。
	  
	  目前 Golang 上游并无好的方式来规避这一问题，而 Uber 提出了一种 Workaround [uber-go/automaxprocs](https://github.com/uber-go/automaxprocs)。利用这一个包，可以在运行时根据 cgroup 或者 runtime 来修改 GOMAXPROCS，来选择一个合适的取值，值得一试。 ([View Highlight](https://read.readwise.io/read/01hg3nj3tpr28yvsjxbrh0bw6s)) #[[golang]] #[[card]]