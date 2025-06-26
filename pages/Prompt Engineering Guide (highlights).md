title:: Prompt Engineering Guide (highlights)
author:: [[promptingguide.ai]]
full-title:: "Prompt Engineering Guide"
category:: #articles
url:: https://www.promptingguide.ai/zh/techniques/cot
summary:: The Prompt Engineering Guide discusses techniques like Chain-of-Thought (CoT) prompting, which improves reasoning by breaking down complex tasks into simpler steps. It also introduces Zero-Shot CoT, where prompts encourage models to think step-by-step, leading to better answers. Lastly, Auto-CoT automates the process of generating reasoning chains, improving efficiency and accuracy in solving problems.
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Jun 16th, 2025]]
	- 链式思考（CoT）提示[](https://www.promptingguide.ai/zh/techniques/cot/#链式思考cot提示)
	  
	  ![COT](https://www.promptingguide.ai/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fcot.1933d9fe.png&w=1920&q=75)
	  
	  图片来源：[Wei等人（2022） (opens in a new tab)](https://arxiv.org/abs/2201.11903)
	  
	  在 [Wei等人（2022） (opens in a new tab)](https://arxiv.org/abs/2201.11903) 中引入的链式思考（CoT）提示通过中间推理步骤实现了复杂的推理能力。您可以将其与少样本提示相结合，以获得更好的结果，以便在回答之前进行推理的更复杂的任务。
	  
	  *提示：*
	  
	    这组数中的奇数加起来是偶数：4、8、9、15、12、2、1。
	    A：将所有奇数相加（9、15、1）得到25。答案为False。
	    
	    这组数中的奇数加起来是偶数：17、10、19、4、8、12、24。
	    A：将所有奇数相加（17、19）得到36。答案为True。
	    
	    这组数中的奇数加起来是偶数：16、11、14、4、8、13、24。
	    A：将所有奇数相加（11、13）得到24。答案为True。
	    
	    这组数中的奇数加起来是偶数：17、9、10、12、13、4、2。
	    A：将所有奇数相加（17、9、13）得到39。答案为False。
	    
	    这组数中的奇数加起来是偶数：15、32、5、13、82、7、1。
	    A：
	  
	  *输出：*
	  
	    将所有奇数相加（15、5、13、7、1）得到41。答案为False。
	  
	  哇！我们可以看到在提供推理步骤时得到了完美的结果。实际上，我们可以通过提供更少的示例来解决此任务，即仅一个示例似乎就足够了：
	  
	  *提示：*
	  
	    这组数中的奇数加起来是偶数：4、8、9、15、12、2、1。
	    A：将所有奇数相加（9、15、1）得到25。答案为False。
	    
	    这组数中的奇数加起来是偶数：15、32、5、13、82、7、1。
	    A：
	  
	  *输出：*
	  
	    将所有奇数相加（15、5、13、7、1）得到41。答案为False。
	  
	  请记住，作者声称这是足够大的语言模型才会出现的新兴能力。
	  
	  零样本 COT 提示[](https://www.promptingguide.ai/zh/techniques/cot/#零样本-cot-提示)
	  
	  ![Zero-shot COT](https://www.promptingguide.ai/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fzero-cot.79793bee.png&w=1920&q=75)
	  
	  图片来源：[Kojima等人（2022） (opens in a new tab)](https://arxiv.org/abs/2205.11916)
	  
	  最近提出的一个新想法是 [零样本CoT (opens in a new tab)](https://arxiv.org/abs/2205.11916)（ Kojima 等人，2022年），它基本上涉及将“让我们逐步思考”添加到原始提示中。让我们尝试一个简单的问题，看看模型的表现如何：
	  
	  *提示：*
	  
	    我去市场买了10个苹果。我给了邻居2个苹果和修理工2个苹果。然后我去买了5个苹果并吃了1个。我还剩下多少苹果？
	  
	  *输出：*
	  
	    11个苹果
	  
	  答案是不正确的！现在让我们尝试使用特殊提示。
	  
	  *提示：*
	  
	    我去市场买了10个苹果。我给了邻居2个苹果和修理工2个苹果。然后我去买了5个苹果并吃了1个。我还剩下多少苹果？
	    
	    让我们逐步思考。
	  
	  *输出：*
	  
	    首先，您从10个苹果开始。
	    您给了邻居和修理工各2个苹果，所以您还剩下6个苹果。
	    然后您买了5个苹果，所以现在您有11个苹果。
	    最后，您吃了1个苹果，所以您还剩下10个苹果。
	  
	  令人印象深刻的是，这个简单的提示在这个任务中非常有效。这在您没有太多示例可用于提示时特别有用。 ([View Highlight](https://read.readwise.io/read/01jxvn00tmey6rs5a0hk077y63))
		- 💡: 好的cot提示词，甚至可以做到zero-shot
- New highlights added [[Jun 20th, 2025]] at 12:36 AM
	- 自动思维链（Auto-CoT）
	  
	  当使用思维链提示时，这个过程需要手工制作有效且多样化的例子。这种手动工作可能会导致次优解决方案。[Zhang et al. （2022） (opens in a new tab)](https://arxiv.org/abs/2210.03493) 提出了一种消除人工的方法，即利用 LLMs “让我们一步一步地思考” 提示来生成一个接一个的推理链。这种自动过程仍然可能在生成的链中出现错误。为了减轻错误的影响，演示的多样性很重要。这项工作提出了Auto-CoT，它对具有多样性的问题进行采样，并生成推理链来构建演示。
	  
	  Auto-CoT 主要由两个阶段组成：
	  
	  •   阶段1：**问题聚类**：将给定问题划分为几个聚类
	  •   阶段2：**演示抽样**：从每组数组中选择一个具有代表性的问题，并使用带有简单启发式的 Zero-Shot-CoT 生成其推理链
	  
	  简单的启发式方法可以是问题的长度（例如，60 个 tokens）和理由的步骤数（例如，5 个推理步骤）。这鼓励模型使用简单而准确的演示。 ([View Highlight](https://read.readwise.io/read/01jy4d2avxwj1gchxz5zcc49ne))
		- 💡: 需要更具体一点的例子，让 gemini 给一个