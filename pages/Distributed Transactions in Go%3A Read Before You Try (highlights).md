title:: Distributed Transactions in Go: Read Before You Try (highlights)
author:: [[Three Dots Labs]]
full-title:: "Distributed Transactions in Go: Read Before You Try"
category:: #articles
url:: https://threedots.tech/post/distributed-transactions-in-go/
summary:: In the previous post, I looked into running transactions in a layered architecture.
Now, let’s consider transactions that need to span more than one service.
If you work with microservices, a time may come when you need a transaction running across them.
Especially if the way they are split was an afterthought (the unfortunate but likely scenario).
Service A calls service B, which calls service C, and if something goes wrong at the end, the system becomes inconsistent.
It would be helpful to have a way to roll back the changes in all the services.

Now, you’re looking at distributed transactions or the saga pattern.
It’s a solved problem. Sometimes, there are good reasons to use them.
But more often, it’s overkill.
If you go down this path, your architecture quickly becomes much more complex than you’d like.
(I learned this the hard way.)
If you need things to be consistent, why split them in the first place? The whole idea of using microservices is to keep independent concepts separate.
Let’s face it: your micr...
![](https://threedots.tech/media/apple-touch-icon.png)

- Highlights first synced by [[Readwise]] [[Oct 8th, 2024]]
	- **Let’s face it: your microservice boundaries are likely wrong if you consider using a distributed transaction.** This is super common and not easy to solve. But it can get much worse if you apply the wrong pattern to the mess you already have. ([View Highlight](https://read.readwise.io/read/01j9kvxzm1ch9jz47c4mhzac8j))