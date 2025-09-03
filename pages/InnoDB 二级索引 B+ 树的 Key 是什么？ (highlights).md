title:: InnoDB 二级索引 B+ 树的 Key 是什么？ (highlights)
author:: [[王炳杰（唤舟）]]
full-title:: "InnoDB 二级索引 B+ 树的 Key 是什么？"
category:: #articles
url:: http://mysql.taobao.org/monthly/2025/07/02/
summary:: 导言
![](http://mysql.taobao.org/monthly/pic/202507/2025-07-25-huanzhou-media/e00041e55d6e9fac20b2136b7bdc7ba30c8d2ff0.png)

- Highlights first synced by [[Readwise]] [[Aug 19th, 2025]]
	- MySQL InnoDB 采用索引组织表存储数据，对于聚簇索引，Key 是主键字段，Value 是行数据。对于二级索引，普遍认知是它包含了二级索引字段和主键字段，但这里存在一个细节问题，二级索引 B+ 树的 Key 是什么？该问题可以从这两个角度展开思考：
	  
	  •   聚簇索引是 unique 的，但二级索引并不一定是，非 unique 的二级索引如何实现？
	  •   InnoDB 使用 MVCC 来避免读写冲突：当一个更新操作只改变了主键的值（pkv，primary key value）而未改变二级索引字段值（skv，secondary key value）的时候，聚簇索引上表现为删除 + 插入，此时二级索引该如何维护呢？update in place 是不合理的
	  
	  实际上，InnoDB 二级索引的 Key 是二级索引字段 + 不在其中的主键字段，这样能简单直接地解决上述问题。 ([View Highlight](https://read.readwise.io/read/01k309kf4x1xq1h2f1j8nwcc5d))