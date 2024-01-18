title:: MySQL · 行业动态 · AWS Re:Invent2023 Aurora 发布了啥 (highlights)
author:: [[baotiao]]
full-title:: "MySQL · 行业动态 · AWS Re:Invent2023 Aurora 发布了啥"
category:: #articles
url:: http://mysql.taobao.org/monthly/monthly/2023/12/01/
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Jan 18th, 2024]]
	- 在分布式事务实现上, 通过EC2 TimeSync service 实现和 Google 的 True Time 类似的解决方案.
	  
	  Ture Time 解决方案核心逻辑是 adding latency in the commit time. 在 Spanner 里面这里叫 commit wait. 等earlist possible time > t110 的时候, 那么就可以确保事务提交了, 这里肯定增加了commit 的时候的 latency, 这里EC2 TimeSync service 越精确, 也就是[earliest possible time, latest possible time] 范围越小, 那么对事务提交的影响是越小的.
	  
	  这里 Aurora limitless 做了优化, commit wait 的时候和 disk IO 是并行的, 由于在寄存分离架构下, disk IO 是网络的 disk IO 需要增加网络的延迟, 这里一般单次 IO 在 tcp 场景下是有可嫩需要 300~400us 左右的. 而 EC2 TimeSync service 保证的精确时间在 us 级别, 那么绝大部分情况下这个时间都可以忽略不计, 因为大部分commit wait 的过程, disk IO 还没有完成, 所以这里可以忽略不计了. ([View Highlight](https://read.readwise.io/read/01hme3r3571yc75ce187hrejdb))
	- Aurora limitless 定位有点尴尬, 不一定能够发展很好. 目前 Aurora limitless 仅仅支持指定shared_key, 对应的 PolarDB-X 同时支持指定 shared_key 以及对用户完全透明无感的分布式, 以及类似的 tidb 支持对用户完全透明无感分布式. ([View Highlight](https://read.readwise.io/read/01hme3sjxwxxfbs6k0xcgkcf1c))