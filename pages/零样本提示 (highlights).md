title:: 零样本提示 (highlights)
author:: [[promptingguide.ai]]
full-title:: "零样本提示"
category:: #articles
url:: https://www.promptingguide.ai/zh/techniques/zeroshot
![](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

- Highlights first synced by [[Readwise]] [[Jun 16th, 2025]]
	- 零样本提示
	  
	  如今，经过大量数据训练并调整指令的LLM能够执行零样本任务。我们在前一节中尝试了一些零样本示例。以下是我们使用的一个示例：
	  
	  *提示：*
	  
	    将文本分类为中性、负面或正面。
	    
	    文本：我认为这次假期还可以。
	    情感：
	  
	  *输出：*
	  
	    中性
	  
	  请注意，在上面的提示中，我们没有向模型提供任何示例——这就是零样本能力的作用。
	  
	  指令调整已被证明可以改善零样本学习[Wei等人（2022） (opens in a new tab)](https://arxiv.org/pdf/2109.01652.pdf)。指令调整本质上是在通过指令描述的数据集上微调模型的概念。此外，[RLHF (opens in a new tab)](https://arxiv.org/abs/1706.03741)（来自人类反馈的强化学习）已被采用以扩展指令调整，其中模型被调整以更好地适应人类偏好。这一最新发展推动了像ChatGPT这样的模型。我们将在接下来的章节中讨论所有这些方法和方法。
	  
	  当零样本不起作用时，建议在提示中提供演示或示例，这就引出了少样本提示。 ([View Highlight](https://read.readwise.io/read/01jxvme2mz96jmp53dc5dw4gys))