title:: Docker容器热迁移CRIU (highlights)
author:: [[Dengguangxing]]
full-title:: "Docker容器热迁移CRIU"
category:: #articles
url:: https://readwise.io/reader/document_raw_content/113010149
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Nov 24th, 2023]]
	- CRIU不支持的应用  使用了以下特性
	  Tasks with debugger attached
	  Task running in compat mode
	  UNIX sockets with relative path
	  Sockets other than TCP, UCP, UNIX, packet and netlink
	  Cork-ed UDP sockets
	  SysVIPC
	  memory segment without IPC namespace
	  ... ([View Highlight](https://read.readwise.io/read/01hg096n5nq115axcxbwrwdj25)) #[[criu]]