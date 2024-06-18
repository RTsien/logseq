title:: 一种更简洁的多模型管理与使用方式 (highlights)
author:: [[飘逝的风]]
full-title:: "一种更简洁的多模型管理与使用方式"
category:: #articles
url:: https://gameapp.club/post/2024-06-10-oneapi-and-models-tips/
summary:: 我之前一篇文章已经介绍过One API了，使用到目前，我很喜欢。喜欢它的统一性，让各种其它LLM的项目轻易的调用后端模型；喜欢它的的UI简洁，模型的使用情况一眼可见。社区更新也比较频繁，新模型支持也挺快。最近我在梳理自己使用AI工具的一些流程，发现有些东西最好再补充一些，于是再更新一篇关于One API的使用技巧。

效果
玩游戏时作为一个收集控，如今各种模型层出不穷，显然阻止不了我的收集心。于是也收藏了一堆，用不用以后再说，拥有的过程就很开心：）

可以看到国内外主流的都纳入囊中啦（个别是借的key）。有这么多模型，使用上比较适合的还是推荐之前介绍过的ChatGPT-Next-Web，主要原因是它支持随时切换模型，并且通知启动参数自定义模型列表。

我可以随时切换到某些模型来场PK（未来有空给他们一个空间竞技）。
本文谈一下部署和使用上的几个问题，众多模型的价格对比也是我之前关注但没有啥概念的。
部署
One API的部署之前文章也提过，这次为了更加正式，我将它部署在我的k8s集群内了，并且通过MySQL持久化，避免数据损失风险。


 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31


apiVersion: apps/v1
kind: Deployment
metadata:
 name: oneapi
spec:
 selector:
 matchLabels:
 app: oneapi
 replicas: 1
 template:
 metadata:
 labels:
 app: oneapi
 spec:
 containers:
 \- name: oneapi
 image: justsong/one-api:v0.6.7-alpha.4 # 视情况选择，我写文章时，用字节豆包这个版本才有
 env:
 \- name: TZ
 value: "Asia/Shanghai"
 \- name: SQL_DSN # mysql使用
 value:
 "root:<mysql-password>@tcp(<mysql-ip>:3306)/oneapi"
 \- name: HTTP_PROXY
 value: "http://<user>:<password>@<proxy-...
![](https://gameapp.club/images/2024/03/oneapi-homepage.webp)

- Highlights first synced by [[Readwise]] [[Jun 17th, 2024]]
	- 公司名称模型名称输入Token价格 (每千Token)输出Token价格 (每千Token)OpenAIGPT-4$0.005$0.015GPT-3.5 turbo$0.0005$0.0015 ([View Highlight](https://read.readwise.io/read/01j0eg1ewsy7kzhsffyrdzj56g))