title:: 【WIP】Dir设计 (highlights)
author:: [[woa.com]]
full-title:: "【WIP】Dir设计"
category:: #articles
url:: https://iwiki.woa.com/p/4012186836
summary:: The text discusses the design of a Dir service for the Soul project, focused on ensuring continuous deployment without downtime. It highlights key features like rolling updates, load balancing, and session management for different types of services, including stateless and stateful services. The Dir service aims to simplify service discovery and improve performance by reducing complexity and memory usage.
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[May 19th, 2025]]
	- 短状态服务
	  
	  •   特点
	    
	    •   状态时间短：无需迁移数据
	        
	    •   路由不变化：只有第一次创建cache，后续复用到相同链路
	        
	    •   自定义负载策略：matchsvr，基于后端负载，优先打满一台；dsroom，基于后端负载，选择最低负载
	        
	    •   分玩法部署：多玩法代码同构，但是小玩法不能影响大玩法
	        
	  •   **目前dirsvr的实现，是完全可以承担** ([View Highlight](https://read.readwise.io/read/01jvk4yyzhqk5m1qc7mt7paqx4))
	- 短状态服务
	  
	  •   特点
	    
	    •   状态时间短：无需迁移数据
	        
	    •   路由不变化：只有第一次创建cache，后续复用到相同链路
	        
	    •   自定义负载策略：matchsvr，基于后端负载，优先打满一台；dsroom，基于后端负载，选择最低负载
	        
	    •   分玩法部署：多玩法代码同构，但是小玩法不能影响大玩法
	        
	  •   **目前dirsvr的实现，是完全可以承担** ([View Highlight](https://read.readwise.io/read/01jvk4zpmhpj2brgddw35jmcz7))