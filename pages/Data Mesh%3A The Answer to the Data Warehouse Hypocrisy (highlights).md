title:: Data Mesh: The Answer to the Data Warehouse Hypocrisy (highlights)
author:: [[Daniel Abadi]]
full-title:: "Data Mesh: The Answer to the Data Warehouse Hypocrisy"
category:: #articles
url:: https://www.starburst.io/blog/data-mesh-the-answer-to-the-data-warehouse-hypocrisy/
![](https://www.starburst.io/wp-content/uploads/2022/05/Data-Mesh-Blog-2.png)

- Highlights first synced by [[Readwise]] [[Dec 4th, 2023]]
	- *Note: I start this piece with some technical background that has nothing to do with the data mesh, and is only relevant to data warehousing insofar as it explains how the parallel database systems that often underlie data warehousing solutions usually work. The main thesis of this article does not begin until the fourth paragraph. But the first three paragraphs serve as important background behind this thesis.*
	  
	  There is one rule, pretty much the only rule, that determines success in scaling data systems: **avoid coordination whenever possible**. This one rule neatly summarizes my entire career until this point — the quarter century I’ve spent teaching, innovating, and building scalable data systems. At the end of the day it all comes down to one thing — avoiding coordination.
	  
	  From a technical perspective, it is obvious why avoiding coordination is so important. No CPU processor can single-handedly handle the resource-intensive workloads that are required in modern data systems. So a scalable system necessarily requires multiple processors, often on the order of thousands or more, typically distributed across multiple independent servers with their own memory and often their own stable storage. When each processor can run completely independently of the other processors, you automatically get linear scalability. If you double the processors, you double the workload the system is able to handle.
	  
	  **  
	  The only thing that gets in the way of linear scalability is coordination**. If one processor needs **anything** from another processor, whether it be data, metadata, lock permission, scheduling permission, or anything at all such that it cannot proceed until it receives what it needs from the other processor, that processor is prevented from being able to continuously make progress, and this introduces scalability bottlenecks. Every system that I’ve been involved in building or designing — from scalable transaction systems such as Calvin/Fauna, H-Store/VoltDB, Bohm, Orthrus, and SLOG to scalable data analytics systems such as C-Store/Vertica, HadoopDB/Hadapt, and Borealis, the primary innovations behind these systems involved making each processor as independent as possible.
	  
	  All of this is why the concept of a “[data warehouse](https://www.starburst.io/learn/data-fundamentals/what-is-data-warehouse/)” is one of the most hypocritical ideas on the face of the planet. Modern data warehouses are typically built using cutting edge scalable software using the architectural principles that have emerged from my lab along with the many other research labs world-wide that are innovating in the area of scalable systems. Tremendous care is taken to achieve near linear scalability of the software, allowing potentially thousands of machines to operate in parallel with minimal communication and coordination. The improvement in the scalability of these systems and the overall progress that has been made in the past few decades has been stunning, dizzying, and inspiring.
	  
	  And yet we go and tell businesses to deploy these super scalable systems inside an organizational structure that is the very antithesis of scalability: the data warehouse. Anybody who has been involved in the deployment of a non-trivial enterprise-wide data warehouse knows that the endeavour is **filled with coordination**. Organizationally, it is a centralized behemoth, a single source of truth. **Centralization and parallelization are antonyms**. Scalability requires independent units working in parallel, while centralization introduces coordination, resistance, and inertia.
	  
	  Getting data into a data warehouse typically requires a great deal of coordination between those in charge of the source system data, those in charge of data governance, data quality, master data management, data integration, those in charge of the data platform, devops, and the data engineers or data scientists that are driving the incorporation of this new data. It is routine to experience delays of multiple-months or longer to get data included in the data warehouse. Making changes to the layout or schema of the data once it is there is a similarly time consuming and coordination-heavy process. Extracting data from the data warehouse — especially when it involves connecting tools to the data warehouse — similarly requires significant amounts of coordination. Running queries over data in the data warehouse are blazing fast and scalable. Everything else organizationally about the data warehouse is slow and unscalable. Is there any wonder why so many data warehouse projects have oversold and under-delivered? How can it be that the world’s experts in scalability can continue to tell customers to deploy their software in such unscalable environments? ([View Highlight](https://read.readwise.io/read/01hgsc2dw31jvda23ejx93wtbr))
		- 💡: Scaling data systems 📈
		  Success determined by rule 📜
		  Avoid coordination 🚫
		  
		  \---
		  
		  “唯一妨碍线性可扩展性的是协调。”
		  
		  ---
		  
		  这就是为什么“[数据仓库](https://www.starburst.io/learn/data-fundamentals/what-is-data-warehouse/)”的概念是地球上最虚伪的想法之一。
		  
		  ---
		  
		  现代数据仓库通常使用最先进的可扩展软件构建，使用从我的实验室以及全球许多其他研究实验室中出现的架构原则，这些实验室正在创新可扩展系统领域。
		  
		  ---
		  
		  “然而，我们却告诉企业在数据仓库这种极不可扩展的组织结构内部部署这些超级可扩展的系统。”
		  
		  ---
		  
		  集中化和并行化是反义词。
	- Getting data into a data warehouse typically requires a great deal of coordination between those in charge of the source system data, those in charge of data governance, data quality, master data management, data integration, those in charge of the data platform, devops, and the data engineers or data scientists that are driving the incorporation of this new data. It is routine to experience delays of multiple-months or longer to get data included in the data warehouse. Making changes to the layout or schema of the data once it is there is a similarly time consuming and coordination-heavy process. Extracting data from the data warehouse — especially when it involves connecting tools to the data warehouse — similarly requires significant amounts of coordination. Running queries over data in the data warehouse are blazing fast and scalable. Everything else organizationally about the data warehouse is slow and unscalable. Is there any wonder why so many data warehouse projects have oversold and under-delivered? How can it be that the world’s experts in scalability can continue to tell customers to deploy their software in such unscalable environments? ([View Highlight](https://read.readwise.io/read/01hgscw2y28sexkadj6jwzwefw))
		- 💡: 将以下段落翻译成中文：
		  
		  """
		  将数据导入数据仓库通常需要在负责源系统数据、数据治理、数据质量、主数据管理、数据集成、数据平台、DevOps 以及驱动新数据合并的数据工程师或数据科学家之间进行大量协调。通常需要数月或更长时间的延迟才能将数据包含在数据仓库中。一旦数据放在那里，对数据布局或架构进行更改同样需要耗费大量时间和协调工作。从数据仓库中提取数据，尤其是涉及将工具连接到数据仓库时，同样需要大量协调。在数据仓库中对数据运行查询非常快速和可扩展。关于数据仓库的其他组织方面都是缓慢和不可扩展的。难怪为什么这么多数据仓库项目都过度销售和交付不足？世界上的可扩展性专家如何能继续告诉客户在这种不可扩展的环境中部署软件？
		  """
	- Below I will summarize the idea, through my own scalability lens. ([View Highlight](https://read.readwise.io/read/01hgsd0x16qbmk6293n22z14jc)) #[[esl]]
		- 💡: 以下我将通过我的可扩展性视角总结这个想法。
- New highlights added [[Dec 4th, 2023]] at 1:35 AM
	- The right way to scale silicon data processing is to **partition the data**. And the right way to scale the human effort in maintaining data sets is to **partition the data**. ([View Highlight](https://read.readwise.io/read/01hgss547gzftzayrmxjstakhb))
	- **The data mesh takes the page directly out of the parallel [DBMS](https://www.starburst.io/learn/data-fundamentals/what-is-dbms-database-management-system/) playbook, and applies it at the business organization level**. Allow teams to develop expertise in particular datasets, and empower them to take full and complete ownership of that data. They bring together datasets that are relevant for their core competency, perform the extractions, transformations, and make the data accessible not only for their own needs, but delivered as a finished product that can be accessed by other teams within the organization as well. ([View Highlight](https://read.readwise.io/read/01hgssbp9ccqwj5kfvp18e3yn8))
	- One key difference between humans and machines is that humans tend to be much more heterogeneous. Each team may have substantially different levels of technical expertise and preferred data management tools. Some teams prefer working with data processing tools such as Spark or Hadoop, other teams with database systems such as MySQL or Oracle, other teams with raw data sitting on a file system. Forcing particular technologies on data mesh teams is a form of coordination that limits their ability to work independently and focus on their core competency. ([View Highlight](https://read.readwise.io/read/01hgssdabhqnqpexwr8hbzfncb))
		- 💡: So, sometimes people work together to collect and use data. But each group might have different ways of doing things and different tools they like to use. It's important to let each group use what works best for them so they can work well and be experts in their own area. If we make everyone use the same tools, it can be harder for them to do their job and they might not be able to focus on what they're best at.
	- The natural outcome of the data mesh is therefore a potpourri of data sets, organized in different formats, stored in different types of systems, located in various public clouds, on premise, or within SaaS systems. Without the right data infrastructure technology, the data mesh will lead to an overwhelming number of [data silos](https://www.starburst.io/learn/data-fundamentals/data-silos/), that makes accessing the data products from other teams inconvenient, intractable, or infeasible. All the good of the rapid progress made possible by giving teams independence and self-service will get repaid with hard walls between data products that are difficult to surmount. This danger of the data mesh is perhaps the biggest reason why the data warehouse has managed to survive for so long, despite the hypocrisy. ([View Highlight](https://read.readwise.io/read/01hgssj0wpb92yz8ffq9xmazme))
	- Accessing data stored outside the system has typically been possible only through slow and cumbersome system extensions such as “external table” mechanisms. ([View Highlight](https://read.readwise.io/read/01hgssmnn2zs8q063bn0fp2wnt))
	- Increasingly, the parallel query processing engine is becoming independent of data storage. In some systems, such as Presto, [Trino](http://trino.io), and Starburst, the query processing engine has become so completely independent that they don’t even come with any data storage whatsoever. The data consumer simply points these systems to data sources — whether they be sitting as Parquet files in S3 object storage, database tables in a [RDBMS](https://www.starburst.io/learn/data-fundamentals/what-is-database/#RDBMS), or an API to a SaaS system — and these systems are able to extract and scalably process data using state-of-the-art parallel processing techniques. ([View Highlight](https://read.readwise.io/read/01hgsspaarra30ya3sx5xk3jb9))
		- 💡: 文章核心内容。其实是个常识。
	- Indeed it is perhaps the most important tool necessary to avoid the silos and chaos that would otherwise arise. ([View Highlight](https://read.readwise.io/read/01hgssx56mazm03rfqdf625jcw))