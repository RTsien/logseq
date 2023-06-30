- ```shell
  # 客户端和服务器连接正常
  
  redis 127.0.0.1:6379> PING
  PONG
  
  # 客户端和服务器连接不正常(网络不正常或服务器未能正常运行)
  
  redis 127.0.0.1:6379> PING
  Could not connect to Redis at 127.0.0.1:6379: Connection refused
  ```