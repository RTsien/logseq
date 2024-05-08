title:: 深入解析go Timer 和Ticker实现原理 (highlights)
author:: [[知乎专栏]]
full-title:: "深入解析go Timer 和Ticker实现原理"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/430429302
summary:: The text explains the implementation principles of Go Timer and Ticker in detail, focusing on how timers are managed and moved between different processing units in Go. It describes how timers are modified, added, and removed in the context of Go's runtime system. The text also covers functions like `moveTimers`, `checkTimers`, and `runtimer` that handle timer operations efficiently in Go's runtime environment.
{:height 248, :width 240}

- Highlights first synced by [[Readwise]] [[May 7th, 2024]]
	- **sendTime实现** #golang #timer
	  ```go
	  //c interface{} 就是NewTimer 赋值的参数，就是channel
	    func sendTime(c interface{}, seq uintptr) {
	        select {
	        case c.(chan Time) <- Now(): //写不进去的话，C 已满，走default 分支
	        default:
	        }
	    }
	  ```
		- sendTime 是不阻塞的，在Timer 实现里面是不会被阻塞的，因为只写一次数据。但是在Ticker里面就会存在阻塞，因为容量为1，ticker 会按时间间隔周期性的写数据到C，这时候如果没有写进去，这次写事件就会丢弃。那么是怎么做到呢？  
		    case c.(chan Time) <- Now() 的时候，如果C 里面的数据没人取走，那么C 已满，case 这条分支发送数据到C就会执行失败而走下面的default。相当于本次调用没有任何操作。
		- 官方注释说：如果reader读C数据慢于第二次向C写数据，那么丢掉这次数据是理想的行为。 ([View Highlight](https://read.readwise.io/read/01hx7102e9eag1zsng6v8ab92m))
			- 💡: 不用担心tick内操作太慢会对ticker本身性能有影响