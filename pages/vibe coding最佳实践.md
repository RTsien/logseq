wiki::
updated:: [[2025-10-16]]
created:: [[2025-10-16]]

- 以[[SmartMoneyTracker]]的开发过程为例
  id:: 68eca069-a166-4ccc-a291-90d5eb3d75d8
	- 第一步：使用gemini deep research生成知识文档和代码实现spec [ref](https://gemini.google.com/app/79727399c2518a11)
		- `如何判断某只股票大资金在撤离` -> `对于A股、美股、港股，分析策略是否不同` -> `补充到最终报告里`
		- `给出代码实现spec，内容为标准markdown格式，全部放置到代码展示框里`
	- 第二步：创建github仓库，并且把报告导出到谷歌文档，再通过谷歌文档下载为markdown文件，文件名改为KNOWLEDGE.md。
	  可以顺便调整文献引用格式：
	  ```
	  @PREREQUISITES.md 将所有文献引用改为下面这种格式：
	  [87](#87)
	  <a id="87"></a>87. 中金：南向流入还有多少空间？ \- 华尔街见闻, accessed October 11, 2025, [https://wallstreetcn.com/articles/3743284](https://wallstreetcn.com/articles/3743284)  
	  ```
	- 第三步：代码实现的spec直接复制粘贴到CODE_SPEC.md文件
	- 第四步：让cloud code/cursor根据KNOWLEDGE.md和CODE_SPEC.md编写README.md。如果是cloud code，随后执行一下/init
	- 第五步：让cloud code/cursor开始实现代码
- #LLM/VibeCoding
-
-