wiki::

- cli 都是基于[nbb-logseq](https://github.com/logseq/nbb-logseq)
	- [logseq-to-markdown](https://github.com/dom8509/logseq-to-markdown) 导出成`(Hugo)`markdown
	  id:: 68e87b81-d704-4c68-84b9-050b56f64abf
		- 作者顺便推荐了自己的[hugo主题](https://github.com/dom8509/hugo-PaperMod) #hugo
		- 这个适合用来通过logseq导出到[[blog]]
	- [logseq-query](https://github.com/cldwalker/logseq-query)  支持内容复杂查询
	- [publish-spa](https://github.com/logseq/publish-spa?tab=readme-ov-file#cli) 官方导出html
	  id:: 68e8ca1d-b3ce-4019-8a57-22ddcb36f1ec
		- https://github.com/logseq/logseq/blob/master/deps/publishing/src/logseq/publishing/db.cljs 是实际导出的时候判断property public=true的地方。如果想改成wiki=true就要打补丁。
-