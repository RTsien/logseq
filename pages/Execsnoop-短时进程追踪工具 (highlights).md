title:: Execsnoop-短时进程追踪工具 (highlights)
author:: [[51cto.com]]
full-title:: "Execsnoop-短时进程追踪工具"
category:: #articles
url:: https://blog.51cto.com/liuzhengwei521/2419139
summary:: execsnoop是一个用于追踪短时进程的工具，可以实时监控进程的exec()行为，并输出进程的基本信息。通过使用execsnoop，可以找到导致系统负载和CPU使用率升高的进程。使用方法是将该工具复制到execsnoop文件并赋予执行权限，然后运行即可。通过示例输出可以看到大量的stress进程在不断启动，导致系统负载和CPU使用率升高。
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Mar 18th, 2024]]
	- execsnoop-专门用于为追踪短时进程（瞬时进程）设计的工具； ([View Highlight](https://read.readwise.io/read/01hs6kksr9nj8dvd1n1zpdhyfk))
	- 它通过 ftrace 实时监控进程的 exec() 行为，并输出短时进程的基本信息，包括进程 PID、父进程 PID、命令行参数以及执行的结果。
	  
	  github地址： [https://github.com/brendangregg/perf-](https://github.com/brendangregg/perf-tools/blob/master/execsnoop) ([View Highlight](https://read.readwise.io/read/01hs6km60mmcyh1ksczv75jehs))