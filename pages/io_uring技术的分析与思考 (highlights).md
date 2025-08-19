title:: io_uring技术的分析与思考 (highlights)
author:: [[Charlie]]
full-title:: "io_uring技术的分析与思考"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/595583437
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Jul 7th, 2025]]
	- IO_URING
	  
	  现有的内核AIO机制经过多年的开发，仍然存在大量的问题。内核维护者认为已有机制问题太多，使用场景太少，早期设计时没有考虑到异步IO的实际需求，在其基础上开发新特性已经没有意义。因此决定针对AIO机制开发中发现的一系列问题，从头开发一套新的异步IO机制，称作io_uring。根据笔者的理解，uring这个朴实的名字就是user和ring的意思，这两者也是io_uring机制的核心。io_uring的高效性就是建立在使用[用户态](https://zhida.zhihu.com/search?content_id=220484963&content_type=Article&match_order=1&q=%E7%94%A8%E6%88%B7%E6%80%81&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTIwNjI4NzUsInEiOiLnlKjmiLfmgIEiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyMjA0ODQ5NjMsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.pESkWizb8rinnR4ncp6NUbZSV6KodMjm3slxVZbQoBI&zhida_source=entity)(user-space)可访问的[无锁环形队列](https://zhida.zhihu.com/search?content_id=220484963&content_type=Article&match_order=1&q=%E6%97%A0%E9%94%81%E7%8E%AF%E5%BD%A2%E9%98%9F%E5%88%97&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTIwNjI4NzUsInEiOiLml6DplIHnjq_lvaLpmJ_liJciLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyMjA0ODQ5NjMsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.STwGXMGaph_9lFaULgyF6pFNtk3L9N6zxNdTtxTb1Vs&zhida_source=entity)(ring)的基础之上的。开发者Jens Axboe在Efficient IO with io_uring中详细介绍了io_uring的设计思想和使用方式，在Linux异步IO新时代：io_uring中对其内容做了概括，【译】高性能异步 IO — io_uring(Effecient IO with io_uring)是其中文翻译（建议参考原文阅读）。 ([View Highlight](https://read.readwise.io/read/01jzjcnxr0h9x9bb5y3q9hvcn7))