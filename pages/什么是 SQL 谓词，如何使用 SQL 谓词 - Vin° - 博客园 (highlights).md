title:: 什么是 SQL 谓词，如何使用 SQL 谓词 - Vin° - 博客园 (highlights)
author:: [[cnblogs.com]]
full-title:: "什么是 SQL 谓词，如何使用 SQL 谓词 - Vin° - 博客园"
category:: #articles
url:: https://www.cnblogs.com/vin-c/p/16371426.html#%E4%B8%80%E4%BB%80%E4%B9%88%E6%98%AF%E8%B0%93%E8%AF%8D
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/logo_square.png)

- Highlights first synced by [[Readwise]] [[Jun 17th, 2024]]
	- 本文将会和大家一起学习 SQL 的抽出条件中不可或缺的工具——**谓词**（predicate）。虽然之前我们没有提及谓词这个概念，但其实大家已经使用过了。例如，`=`、`<`、`>`、`<>` 等比较运算符，其正式的名称就是比较谓词。 #card
	  id:: 67df0c90-d219-43d7-a543-ccc27555eefa
		- 通俗来讲谓词就是 [SQL 常用的函数](https://www.developerastrid.com/sql/sql-commonly-used-functions/) 中介绍的函数中的一种，是需要满足特定条件的函数，该条件就是返回值是真值。
		- 对通常的函数来说，返回值有可能是数字、字符串或者日期等，但是谓词的返回值全都是真值（`TRUE`/`FALSE`/`UNKNOWN`）。这也是谓词和函数的最大区别。 ([View Highlight](https://read.readwise.io/read/01j0e5frjghsb0g3ws2fbwx8e3))