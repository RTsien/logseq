title:: 有坑勿踩（二）——关于游标 (highlights)
author:: [[segmentfault.com]]
full-title:: "有坑勿踩（二）——关于游标"
category:: #articles
url:: https://segmentfault.com/a/1190000017564466
summary:: This text discusses the use of cursors in MongoDB and highlights common pitfalls, especially regarding the `toArray()` method, which can cause memory issues if used carelessly. It explains that cursors are used for data retrieval during queries and can time out if not accessed within a certain period. The author advises on using methods like `hasNext()` and `next()` for efficiency and warns against disabling cursor timeouts.
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/touch-icon.png)

- Highlights first synced by [[Readwise]] [[Oct 5th, 2024]]
	- 在游标超时的时候你得到的实际是“游标不存在”错误，而不是超时。那么反过来是不是也成立呢，“游标不存在”一定是超时了吗？离散数学告诉我们，一个命题的逆命题不一定成立。事实上也是如此。“游标不存在”的另一种可能性是有些用户热衷于在MongoDB前面加上负载均衡/自动故障恢复的软/硬件。我们已经知道游标是存在于一台服务器上的，如果你的负载均衡毫无原则地将请求转发到任意服务器上，`getmore`同时会因为找不到游标而出现“游标不存在”的错误。  
	  事实上MongoDB和其驱动本身就已经能够完成高可用和负载均衡，并不需要额外画蛇添足。 ([View Highlight](https://read.readwise.io/read/01j8m0rnkj64cn3b9rwehjk6xn))