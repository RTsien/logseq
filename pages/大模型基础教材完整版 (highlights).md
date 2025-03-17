title:: 大模型基础教材完整版 (highlights)
author:: [[readwise.io]]
full-title:: "大模型基础教材完整版"
category:: #articles
url:: https://readwise.io/reader/document_raw_content/257650070
summary:: The text lists several academic papers related to machine learning and language models. Key works include a study on transfer learning with a unified text-to-text transformer and efficient fine-tuning methods using residual learning. These studies explore advancements in natural language processing techniques.
![](https://readwise-assets.s3.amazonaws.com/media/reader/parsed_document_assets/257650070/QmQCzux_N-WAHLcZ51xlfe2arY5M2lTQCvDe2TOApPw-cove_W2wJKXR.png)

- Highlights first synced by [[Readwise]] [[Mar 17th, 2025]]
	- 2 ([View Highlight](https://read.readwise.io/read/01jphbrymaby9vxww7tqqm5c8q))
	- 第 1 章 语言模型基础 优势。但是，这种泛化能力会随着n的增大而逐渐减弱。应用 trigrams 对文本“长 颈鹿脖子长”出现的概率进行计算，将出现以下“零概率”的情况： Ptrigrams(长颈鹿,脖子,长) =
	  C(长颈鹿,脖子,长) C(长颈鹿,脖子)
	  = 0。 (1.4)
	  因此，在 n-grams 语言模型中，n 代表了拟合语料库的能力与对未知文本的泛化能 力之间的权衡。当 n 过大时，语料库中难以找到与 n-gram 一模一样的词序列，可 能出现大量“零概率”现象；在n过小时，n-gram 难以承载足够的语言信息，不足 以反应语料库的特性。因此，在 n-grams 语言模型中，n 的值是影响性能的关键因 素。上述的“零概率”现象可以通过平滑（Smoothing）技术进行改善，具体技术 可参见文献 [11]。 ([View Highlight](https://read.readwise.io/read/01jphbt26jrgwd3swpd7ct3efr))