title:: PolarDB · 引擎特性 · BLOB 实现与性能优化 (highlights)
author:: [[谢榕彪（归墨）]]
full-title:: "PolarDB · 引擎特性 · BLOB 实现与性能优化"
category:: #articles
url:: http://mysql.taobao.org/monthly/monthly/2024/10/01/
summary:: ​ Blob (binary large object) 是 Innodb 中的一种大对象存储类型，既可以存储字符对象，也可以存入二进制对象，在需要存储空间需求较大的数据的场景下，应用非常广泛。
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Nov 4th, 2024]]
	- 当修改长度小于 100 字节时（small change），全局 Lob version 不变，将修改前后完整 blob 内容的 diff 直接记录在 undo log，因此 blob page chain 的内容可以直接原地修改。当读取时，直接从 blob page chain 中获得到最新的 blob 数据后，所访问的旧版本基于 undo log 来重构。 ([View Highlight](https://read.readwise.io/read/01jbsqsvkett1ehf11rmwcswrm))