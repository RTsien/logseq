title:: 对外二测进展与风险 (highlights)
author:: [[woa.com]]
full-title:: "对外二测进展与风险"
category:: #articles
url:: https://iwiki.woa.com/p/4012570189
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Sep 23rd, 2024]]
	- 当前主要风险
	  
	  基础业务风险：较低  
	  系统健壮性风险：测试结论低，但我们仍要进一步保障。开发侧风险需要更明确。  
	  大型玩法风险：较高，需要进入丰富和成熟，也希望有专项的session-review。  
	  工具流程风险：基础发布较低；灰度风险高，要尽快收敛和更多的验证灰度能力。
	  
	  内部测试目标
	  
	  •   获得玩家参考模型，借由数据重新评估一下正式对外实例数，DB承载等。
	    
	  •   各个系统可观测能力，修复和完善缺失部分。增加灰度等可视化，玩法分流可视化
	    
	  •   排查潜在风险，error等。
	    
	  •   本周三下班前推进大型玩法正常循环跑通。
	    
	  •   演练几个部分，先在模拟环境演练，之后放内部测试环境演练。
	    
	    •   灰度（配置、程序、zone）
	        
	    •   容灾测试，引入延迟和故障，观测系统。
	        
	  
	  其它能力工具扩展
	  
	  •   提交合并校验工具
	    
	  •   tool集成常用工具(redis/mongo client)
	    
	  •   red-tool丰富，脱离GM完成绝大多数功能
	    
	  
	  常态化的模拟环境
	  
	  •   融入发布更新流程
	    
	  •   融入日常故障注入
	    
	  •   结合监控告警
	    
	  
	  常态化压测及数据收集
	  
	  建立比较完善的视图，目标用于及早发现性能降低的修改点。
	  
	  主要解决方案
	  
	  1.  繁琐事项文档化checklist，逐步完善和补充。
	    
	  2.  尽早补充研发全链路工具，从提交、到发布、到线上处置等，避免漏洞。
	    
	  3.  常态化演练，重点是融入到日常治理中，融入到日常流程中。 ([View Highlight](https://read.readwise.io/read/01j8ecbtsgfm0twjnbxxq4epgg))