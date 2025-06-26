title:: BatchCommit 实现存在原子性漏洞 (highlights)
author:: [[woa.com]]
full-title:: "BatchCommit 实现存在原子性漏洞"
category:: #articles
url:: https://iwiki.woa.com/p/4012286678
summary:: The BatchCommit implementation has a flaw that allows for unlimited theft of items due to improper error handling during updates. If an update fails, the process stops without rolling back previous changes, resulting in unintended item gains. The proposed fix is to ensure all checks are completed before any updates are made, and to adjust the order of item processing to prevent errors.
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Jun 20th, 2025]]
	- 解决方法
	  
	  理论上，道具增减的检查操作都应该在 CheckItems 中完成，包括：数量合法性、配置合法性、是否已经拥有等等检查，在 UpdateItems 中不应该出现错误，只进行数据操作。
	  
	  这样对业务系统实现道具类型是提出要求，此前并没明确，也没有从机制上暴露出来。
	  
	  1.  UpdateItems 仍然保留错误返回值，但出现错误时会 DPanic 日志暴露，推动业务 owner 修复
	    
	  2.  UpdateItems 返回错误时，不再中断流程，而是 continue 处理其他道具类型的增减
	    
	  
	  ![](https://iwiki.woa.com/tencent/api/attachments/s3/url?attachmentid=22778494)
	  
	  在 zone 内部管理的道具系统都能够用上述方法解决，但的确存在 UpdateItems 中出现 CheckItems 中未暴露的错误的情况，比如说：钻石，可能访问米大师网络抖动或者米大师平台故障等。
	  
	  对此，解决方法是：将钻石放在首个 UpdateItems 的道具类型，基于此前的道具类型优先级机制： ([View Highlight](https://read.readwise.io/read/01jy4j19z9prr3tzndmj9paejs))