- abbreviation for **partial resynchronizations**
- psync参数与返回 #card
  id:: 6497f9c1-a81c-41a0-b0f7-50dd31d0d8e5
	- runID，每个 Redis 服务器在启动时都会自动生产一个随机的 ID 来唯一标识自己。当从服务器和主服务器第一次同步时，因为不知道主服务器的 run ID，所以将其设置为 "?"。
	  > Every Redis master has a replication ID: it is a large pseudo random string that marks a given story of the dataset. 
	  > 官方文档称为replication ID
	- offset，表示复制的进度，第一次同步时，其值为 -1。
	- 主服务器收到`psync`命令后，会用 `FULLRESYNC` 作为响应命令返回给对方。并且这个响应命令会带上两个参数：主服务器的 runID 和主服务器目前的复制进度 offset。从服务器收到响应后，会记录这两个值。
-