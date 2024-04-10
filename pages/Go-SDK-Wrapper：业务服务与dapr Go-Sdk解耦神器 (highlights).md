title:: Go-SDK-Wrapper：业务服务与dapr Go-Sdk解耦神器 (highlights)
author:: [[woa.com]]
full-title:: "Go-SDK-Wrapper：业务服务与dapr Go-Sdk解耦神器"
category:: #articles
url:: https://km.woa.com/articles/show/581881
summary:: The text discusses the development and use of the go-sdk-wrapper tool to simplify and decouple business services from the CNCF Dapr framework. The tool aims to reduce code redundancy, improve maintainability, and enhance readability for business developers working with Dapr APIs. It focuses on adapting RPC, event handling, and message subscription capabilities while abstracting away the complexities of Dapr server integration.
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- 最早由微软孵化的[CNCF Dapr Serverless](https://landscape.cncf.io/serverless)产品，主要以sidecar的方式，不限语言、不限环境、不限中间件的标准化能力，给业务方与第三方依赖完全解耦。并简单地同时以HTTP、gRPC两种方式为业务提供服务，让业务开发者只关注业务需求本身，这是一种重sidecar，轻业务逻辑服务的模式。sidecar由云厂商或者第三方云聚合厂商提供sidecar的可靠性、稳定性、以及可维护性，包括升级的无感知性。目前Dapr在CNCF处于孵化中，主要贡献成员来自于一家公司[diagrid](https://www.diagrid.io/), 成员全部来自于微软Azure团队。 ([View Highlight](https://read.readwise.io/read/01hv1t9d1saha28zr16a62r4cx))