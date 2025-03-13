title:: Rag 的总结整理 (highlights)
author:: [[Anooyman]]
full-title:: "Rag 的总结整理"
category:: #articles
url:: https://mp.weixin.qq.com/s/o6oDamaD2zIruyZFP_erLQ
summary:: RAG（检索增强生成）是一种结合检索和生成技术的AI方法，能够提高自然语言问答的准确性和可靠性。它的优势包括灵活的知识更新和强大的可扩展性，但也依赖于文档质量，可能导致不准确的回答。RAG在处理复杂任务时表现出色，但需优化模块和检索后处理以提升整体效果。
![](https://mmbiz.qpic.cn/mmbiz_jpg/PylicKiadJA3d7Fypiay2ibmFp7ZUYgGIUOQ5XykOw8HBmlT92HUiawcbrERUO51kWj7TLfqVV23uycT9JvWgibJk3ww/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Feb 11th, 2025]]
	- RAG，全称为 Retrieval-Augmented Generation，即检索增强生成。它是一种结合了检索和生成的技术方法，将传统的基于检索的问答系统和基于自然语言生成的技术相结合，提升了 AI 系统在回答自然语言问题时的准确性和可靠性。
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_png/PylicKiadJA3d7Fypiay2ibmFp7ZUYgGIUOQUTGKXajBpmyNgEbPCV4ibty5rOZ1AA5iaCvjdL7qx16rEiaFtET1ydg4A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) ([View Highlight](https://read.readwise.io/read/01jkhj08nxa8kzdnzx8kkgcjvy))
	- 优势
	  
	  RAG 具有多方面的显著优势，使其在自然语言处理领域中占据重要地位。
	  
	  \- 知识更新灵活性高。与传统的微调方法相比，RAG 无需对整个模型进行大规模的重新训练，只需更新知识库中的数据，就能让模型获取到最新的知识信息。例如，在金融领域，市场数据和法规政策不断变化，RAG 系统可以及时将新的股票行情、政策法规等信息纳入知识库，使模型能够快速适应这些变化，为用户提供基于最新信息的准确回答。
	  
	  \- 可扩展性强。RAG 能够轻松应对大规模数据的处理需求，随着检索语料库规模的不断扩大，其性能不会受到明显影响。这是因为它可以灵活地从海量数据中检索出与问题相关的信息，而无需像一些传统模型那样在数据量增加时面临性能瓶颈。以大型电商平台的客服系统为例，随着商品种类和用户数量的不断增加，相关的知识库也在持续扩充，但 RAG 系统依然能够高效地检索和利用这些知识，为用户提供精准的购物咨询服务。
	  
	  \- RAG 在处理复杂任务和开放领域问题时表现出色。它能够从广泛的知识源中检索信息，为模型提供丰富的上下文，从而更好地理解和处理复杂的自然语言任务。无论是多轮对话、长篇文档的理解与生成，还是涉及多个领域知识的综合性问题，RAG 都能通过检索相关信息，为生成准确、全面的回答提供有力支持。例如，在智能写作助手应用中，当用户需要撰写一篇关于科技发展趋势的文章时，RAG 可以从众多的科技文献、新闻报道、行业分析等资料中检索相关信息，并整合到生成的文章中，使文章内容更加丰富、有深度。 ([View Highlight](https://read.readwise.io/read/01jkhj3qznbpc78ns7hegp5xrn))
	- 不足
	  
	  \- RAG 对文档质量的依赖程度较高。如果知识库中的文档内容不准确、过时、存在噪声或格式不规范，将会直接影响检索的准确性和生成答案的质量。例如，在一个技术文档知识库中，如果部分文档存在错误的技术参数或过时的技术描述，RAG 系统在检索和利用这些文档时，可能会生成错误的技术解答，误导用户。而且，文档的切分粒度也会对模型性能产生影响，如果切分不当，可能导致信息碎片化或关键信息被分割在不同的块中，影响检索效果。
	  
	  \- RAG 还可能产生不准确的回答。即使检索到的文档本身信息准确，但由于模型在整合和生成答案过程中的局限性，仍然可能出现回答不准确的情况。例如，当需要对检索到的多个信息片段进行综合推理和判断时，模型可能会出现失误，导致生成的答案与实际情况不符。此外，如果检索到的文档与问题的相关性判断不准确，也会使生成的答案偏离主题或无法满足用户的需求。 ([View Highlight](https://read.readwise.io/read/01jkhj5rzhrwztgvw4a8r86s1g))
	- 模块化RAG的优化
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_png/PylicKiadJA3d7Fypiay2ibmFp7ZUYgGIUOQm5X8I2txSO93fHicEmvicBljGaR9QkFL5HxiadonvSic6PnkgDgA9icIiaMw/640?wx_fmt=png&from=appmsg) ([View Highlight](https://read.readwise.io/read/01jkhj7cpxkedcthmvb32wtfja))
	- Chunk 优化
	  
	  \- 在进行 Chunk 优化时，滑动窗口的设置需要综合考虑多方面因素。如果滑动窗口设置过大，虽然能包含更多的上下文信息，但也可能会引入过多的无关信息，增加检索的负担和噪声；反之，若设置过小，则可能导致信息碎片化，使关键信息被分割在不同的 Chunk 中，影响检索的准确性。例如，在处理一篇科技文献时，如果滑动窗口过大，可能会将多个不同主题的段落包含在一个 Chunk 中，导致检索时相关性判断不准确；而如果过小，可能会把一个完整的实验步骤或理论阐述分割开。因此，需要根据数据的特点和具体的应用场景，通过实验和分析来确定合适的滑动窗口大小，以平衡信息完整性和检索效率 ([View Highlight](https://read.readwise.io/read/01jkhj9gacrayxgk0fffd2dwjr))
	- Metadata 增强主要有两种方式。一种是直接丰富向量数据库中的 metadata 字段，这种方式可以通过添加诸如数据来源、创建时间、作者等信息，在检索时利用多路检索或混合检索技术，更精准地筛选出符合特定条件的信息。例如，在一个新闻资讯的 RAG 系统中，可以将新闻的发布时间、所属类别等信息作为 metadata，当用户查询特定时间段或特定主题的新闻时，能够快速定位到相关的 Chunk。另一种方式是将需要的信息直接加到文本中，适用于在文本被切分后可能出现信息丢失的情况，如后文使用缩写或代词指代前文内容时。通过将原始信息补充完整，可以提高检索的准确性。例如，一篇医学文献中，首次提到 “新冠病毒（COVID - 19）”，在后续的 Chunk 中可能仅使用 “病毒” 一词，那么在 Chunk 划分时将 “新冠病毒” 的完整信息添加到后续相关 Chunk 中，能避免检索时因信息缺失而导致的误判 ([View Highlight](https://read.readwise.io/read/01jkhjb2x71ay6938r44pwb5tz))
	- 知识图谱存储则是将数据以图结构的形式存储在数据库中，如微软提出的 GraphRAG。通过图检索，可以利用实体之间的关系快速找到相关信息。例如，在一个电影知识图谱中，通过演员、导演、电影类型等实体之间的关联关系，可以快速检索到某个演员参演的所有电影、某个导演执导的作品以及具有特定类型的电影集合等。然而，这种存储方式的代价高昂，无论是数据的存储成本还是检索时的计算成本都较高。后续虽有提出 light GraphRAG 等优化方案，但仍无法彻底解决成本问题。并且，对于复杂的知识图谱，数据的更新和维护也面临挑战，因为一个实体或关系的变化可能会影响到整个图谱的结构和语义。 ([View Highlight](https://read.readwise.io/read/01jkhjq46avmj63p1zngsk10cc))
	- PS. 业界有一个观点认为图数据并不一定非要存储在图数据库中，有很多公司在这个方面做过尝试了。现在一个比较好的解决方法是将图数据解析为一条条的chunk 然后存储在向量数据库中，可以通过Meta字段增加一些必要的信息 ([View Highlight](https://read.readwise.io/read/01jkhjqv1hzyrt7rsgep9weena))