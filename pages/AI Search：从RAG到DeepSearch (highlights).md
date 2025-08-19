title:: AI Search：从RAG到DeepSearch (highlights)
author:: [[woa.com]]
full-title:: "AI Search：从RAG到DeepSearch"
category:: #articles
url:: https://km.woa.com/articles/show/633081?kmref=vkm_discover
summary:: AI Search uses retrieval and reasoning to help large language models overcome their limited knowledge and abilities. It evolved from simple retrieval methods to intelligent agents that can think deeply and use tools autonomously. The future goal is to build general AI agents that can solve complex tasks by combining knowledge, reasoning, and tool use.
![](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

- Highlights first synced by [[Readwise]] [[Jul 11th, 2025]]
	- 随着推理模型的出现和发展壮大，RAG从逐渐升级到Agentic RAG，即从之前的人为设计好的固定工作流程慢慢进化到更加自主的智能体系统。智能体自主性的第一个层面是直接根据当前的上下文信息经过推理（深度思考）然后直接决定下一步执行检索的问题和检索策略。其中可用或者建议的用户问题优化策略和可用的检索配置都直接通过上下文信息暴露给模型，模型在深度思考阶段推理和决定合适或者最佳的优化策略，即利用模型强大的推理能力（深度思考能力）自主地完成优化。第二个层面是模型可以根据之前的执行结果自主地判断是否还需要继续搜索以及还有哪些需要进一步探索的缺失之处，这样整个智能体系统可以循环多次搜索步骤直到收集到足够的上下文信息来回答用户最初的问题。
	  
	  Agentic RAG的整体框架本质上其实是推理模型兴起之前的ReAct框架（[ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)），而现如今有了推理模型的加持。不仅如此，最近非常流行的DeepSearch和DeeppResearch也主要基于ReAct框架。其实无论是最开始的朴实RAG，还是如今的Agentic RAG抑或DeepSearch和DeepResearch，都可以统称为AI Search，即通过AI技术来搜索外部知识来为LLM（更好地）回答用户问题补充上下文信息。Jina AI也在博客《[A Practical Guide to Implementing DeepSearch/DeepResearch](https://jina.ai/news/a-practical-guide-to-implementing-deepsearch-deepresearch)》中指出：其实DeepSearch和DeepResearch都算是对RAG的“品牌再造”（rebranding），即一种商业手段。 ([View Highlight](https://read.readwise.io/read/01jzwpx3hgvhzvd9vcjvgy7yb2))
	- DeepSearch
	  
	  有了信息和工具之后，再加上一个智慧大脑（推理模型），我们就很自然地得到了深度搜索（DeepSearch）。目前DeepSearch或者DeepResearch的论文和应用有很多([OpenAI Deep Research](https://openai.com/index/introducing-deep-research/)、[Anthropic Multi-Agent Research System](https://www.anthropic.com/engineering/built-multi-agent-research-system)、[JinaAI Deep(Re)Search Guide](https://jina.ai/news/a-practical-guide-to-implementing-deepsearch-deepresearch)、[Kimi-Researcher](https://moonshotai.github.io/Kimi-Researcher/)、[ByteDance DeerFlow](https://deerflow.tech/)、[Google Gemini Search Agent](https://github.com/google-gemini/gemini-fullstack-langgraph-quickstart)等），总体的技术范式和实现大同小异，与前文所讲的Agentic RAG基本一致，这里不再一一赘述。 ([View Highlight](https://read.readwise.io/read/01jzwq36fxcqk7mb9b1v06v2c3))
	- Jina AI和Google Gemini的方案，二者整体都很简洁，主要面向AI Search垂直领域。整个智能体系统的核心仍然是推理模型及其深度思考能力，具体在AI Search领域，该深度思考能力用来思考判断当前上下文是否足够回答用户问题以及如果不够的话生成后续的检索问题（即gap问题）。除此之外，AI Search通常只有一个检索工具，用来获取外部知识。
	  
	  [Jina AI深度搜索的实现](https://mp.weixin.qq.com/s/wnUZE3ege_o8gBlJteV13Q)
	  
	  [Google Gemini Search Agent框架示意图](https://github.com/google-gemini/gemini-fullstack-langgraph-quickstart)
	  
	  ![](https://km.woa.com/asset/00010002250700ba5a22a3d418438802?height=745&width=1280&imageMogr2/thumbnail/1540x>/ignore-error/1)
	  
	  ![](https://km.woa.com/asset/00010002250700c0cbc855af4a4c5901?height=1474&width=959&imageMogr2/thumbnail/1540x>/ignore-error/1)
	  
	  如果再加入和拓展一些信息检索之外的工具，比如代码解释器、计算机系统或软件控制等工具，那么就可以得到Coding Agent（比如[Cursor](https://cursor.com/)、[Trae](https://www.trae.ai/)、[Cline](https://cline.bot/)、[Github Copilot](https://github.com/features/copilot)等）、Browser Agent（比如[Fireworks Broswer Agent](https://fireworks.ai/blog/opensource-browser-agent)）、Computer Agent（比如[OpenAI Computer Use](https://platform.openai.com/docs/guides/tools-computer-use)和[Anthropic Computer Use](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/computer-use-tool)）等垂直应用Agent。
	  
	  总的来说，AI Search是其他垂直AI应用Agent的前置依赖和基础。同理，以人类为例，阅读是创作的基础和前提。具体地说，AI Search主要用来阅读（read），而其他智能体（比如Coding Agent）的主要目的是创作（write）。LangChain 的博客《[How and when to build multi-agent systems](https://blog.langchain.com/how-and-when-to-build-multi-agent-systems/)》中提到以阅读为主的多智能系统通常比以创作为主的多智能体系统更加容易些。这个很好理解，普通人通常可以阅读四大名著，钻研深入的读者也理解更深度些，但是现实是绝大多数人是写不出来类似四大名这种经典作品的。原创或者自研比阅读和使用要高几个维度。类似地，生成对抗网络（GAN）中生成器（generator）通常比判别器（discriminator）的参数量更大，一个小的LLM可以用来评估另外一个参数量更大的LLM的生成内容。
	  
	  Vertical AI Agents
	  
	  Research Agent
	  
	  Coding Agent
	  
	  Browser Agent
	  
	  Computer Agent
	  
	  ...
	  
	  Search Agent
	  
	  General Agent
	  
	  从工作的角度来看，阅读（学习）不是目的，创作（干活）才是。创业公司[Cognition AI](https://cognition.ai/)发布的[DeepWiki](https://deepwiki.com/)产品支持对一整个代码仓库进行问答甚至DeepResearch，即让AI来帮助我们进行代码阅读和理解，即AI Search。同时这个创业公司发布的Coding Agent产品[Devin](https://devin.ai/)号称为全球首个AI软件工程师，即让AI充当或者替代人类去编写软件，即AI Coding。 ([View Highlight](https://read.readwise.io/read/01jzwq2xshmxv8y2f2r0p4b37g))
	- 智能体系统自主性面临着2个边界问题，即知识边界和能力边界，也分别对应LLMs应用的2大抓手：信息和工具。从目前来看，大（语言）模型仍然是数据驱动的，所以要想解决这2个边界问题，仍然需要从训练数据着手，进行针对性的模型训练。同时考虑到信息空间和工具库都是不断扩展的，环境是动态的（dynamic），并且与环境交互是多轮或多步的（long horizon），目前的训练方式也从有监督微调（SFT）逐步升级进化到强化学习（RL）后训练，以获得更好的自主性、泛化性、灵活性，比如[Search-R1](http://arxiv.org/abs/2503.09516)、[ReSearch](http://arxiv.org/abs/2503.19470)、[Kimi-Researcher](https://moonshotai.github.io/Kimi-Researcher/)、[SimpleTIR](https://simpletir.notion.site/report)、[Alibaba Web Agent系列](https://github.com/Alibaba-NLP/WebAgent)等。正所谓授人以鱼不如授人以渔，SFT 更像是前者的授人以鱼，而RL更像是后者的授人以渔。 ([View Highlight](https://read.readwise.io/read/01jzwq5cryc7ggjta1zd13a4jt))