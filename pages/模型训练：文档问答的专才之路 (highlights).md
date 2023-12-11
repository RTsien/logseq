title:: 模型训练：文档问答的专才之路 (highlights)
author:: [[woa.com]]
full-title:: "模型训练：文档问答的专才之路"
category:: #articles
url:: https://km.woa.com/articles/show/582375
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Nov 29th, 2023]]
	- 常用的检索方法包括传统的倒排索引检索和基于embedding的语义检索。传统倒排索引检索通常采用分词和停用词过滤，然后构建相应的倒排索引。通过基于token粒度的匹配召回，倒排索引检索系统具有速度快、精度高的优势。但由于严格匹配的逻辑，其无法捕获语义上的相似性，泛化能力较差。 ([View Highlight](https://read.readwise.io/read/01hgdp8tgx33rzwsshkbnt7jzz))
	- 随着embedding的普及，主流的检索召回方法已逐步被基于语义embeding的模型方案所取代。基于语义embedding的检索模型通常采用双塔模型结构，其中一个塔用于编码query信息，另一个塔用于编码document信息，通过query-document的pair对数据进行训练。 ([View Highlight](https://read.readwise.io/read/01hgdp8f6xkbdss89yhr09wpjk))
	- 基于语义embedding的检索已成为当下的主流，因此我们首先尝试了基于语义embedding模型的方案。由于缺乏大量的query-document数据对，这里我们主要对比了开源和内部相关团队的几种语义embedding，最终，我们选用了在构造的域内数据集上效果最优的基于sentence_transformers框架的多语言语义embedding模型。在实际应用过程中我们发现基于语义embedding的检索系统虽然召回率较高，但同时也存在明显的误召回和不召回现象。一方面，语义embeding由于其领域不匹配等原因存在召回的内容完全不相关的问题，针对这类问题我们采用基于jaccard距离的后处理方式进行优化，recall@4得到了显著地提升；另一方面，在技术文档问答场景下，我们发现很多的线上问题由技术类关键词组成，这些词的语义embedding效果通常不佳，因此我们引入倒排索引机制优化这类通用问题，通过融合两路召回的方式进一步提升了召回效果；整体上，通过引入jaccard距离优化语义embedding的领域不匹配问题和引入倒排索引融合召回优化专有名词不召回的问题，在我们构造的域内数据集上recall@4指标都有了一定程度的提升，具体效果如下表格所示： ([View Highlight](https://read.readwise.io/read/01hgdpa8bcgt13p1atwd33pvr8))
	- 在一定程度上解决了检索系统的效果问题之后，便可以基于检索得到的相关文档和query构造大模型对应的prompt/instruction输入进而得到我们最终的response。那么如何优化我们的大模型效果使得更适配我们的应用场景，精心优化prompt也会有一定的效果，但很难完全满足业务要求。领域数据微调相对prompt优化可以进一步提升模型在文档问答场景的效果，bert时代采用的是预训练+微调的训练范式，llm时代采用的是基座 + instruction finetune的训练范式。其中基座模型我们采用了组内基于llama中文化的KaLM系列模型（详情可以参考[《OpenKaLM：可商用大模型OpenLLaMA的中文化适配之路》](https://km.woa.com/articles/show/580394)）和当前中文领域最热门的ChatGLM基座模型, 通过对基座模型进行低资源LoRA finetune和全参数FFT(full finetune)对比了两类基座模型在智能客服场景下的应用效果。下面我们会从训练数据集构造以及两类精调训练方式上进行阐述，最后对比不同基座不同训练方式的效果。 ([View Highlight](https://read.readwise.io/read/01hgdpc4fk8padm11zfnqm55qq))
	- 与BERT时代大多数采用全参数finetune范式不同，大模型时代的基座模型规模较大，全参数finetune带来了训练效率低、成本高和验证周期长的问题。为应对这些挑战，一系列高效的低资源方案被开发出来，以适配大模型在下游任务上的应用，各类方案的详细阐述可参考[《大模型低资源精调：单卡一小时就能训练一个专属于你的ChatGPT》](https://km.woa.com/articles/show/576749)一文。在本项目中，我们主要尝试了低资源LoRA finetune方案，LoRA（Low-Rank Adaptation）是一种高效且低资源的训练策略，其核心思想相对简单，如下图所示：
	  
	  **![](https://km.woa.com/asset/ae2507624a9c444a81b8e449e6485a46?height=300&width=333)**
	  
	       通过在原有模型参数W的旁边外接一个旁路，将原始的输入维度d先降维到r，然后再升维度到原始的输出维度d。借此，原有的模型参数量d*d被降维成了d*r + r*d(r<<d), 其中中间维度r被称为参数W的内在秩，基于lora的低资源训练方案在多个应用场景下被证明是有效的。 ([View Highlight](https://read.readwise.io/read/01hgdpf2ev23m654yd2qen18dc))
		- 💡: 这个图太形象了
	- 低资源训练有效解决了大模型训练成本高和训练难的问题，但是另一方面基于低资源的训练方案存在参数量少容易过拟合，未对模型全量参数更新无法充分适配当前场景等问题，因此我们也对大模型进行了Full finetune从而更加全面充分的对比各种训练方式对模型效果的影响，Full finetune的示意图如下所示：
	  
	  ![](https://km.woa.com/asset/e93cd86711864709865bac83d43ed30f?height=514&width=1024)
	  
	       与基于LoRA的低资源训练方式不同，Full finetune需要对全量参数进行正反向传播，消耗的显存大速度慢，本项目结合deepspeed实现对基座大模型的分布式训练。 ([View Highlight](https://read.readwise.io/read/01hgdphx9ya7fzq2tdfs881rm8))