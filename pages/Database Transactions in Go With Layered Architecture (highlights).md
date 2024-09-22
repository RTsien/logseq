title:: Database Transactions in Go With Layered Architecture (highlights)
author:: [[Three Dots Labs]]
full-title:: "Database Transactions in Go With Layered Architecture"
category:: #articles
url:: https://threedots.tech/post/database-transactions-in-go/
summary:: As I join a new company, I often feel like an impostor.
After all the interviews, they really seem to know what they’re doing.
I’m humbled and ready to learn from the best.
On one such occasion, a few days in, I dealt with a production outage and asked the most senior engineer for help.
They came to the rescue and casually flipped a value in the database with a manual update. 🤯
The root cause was that a set of SQL updates were not done within a transaction.
Suddenly, I regretted not asking for higher compensation right away.
Onboarding is fun. What I learned this way is that even if something seems like a fundamental concept (e.g., SQL transactions), it may often be overlooked.
SQL seems like something we all know well, and there are few surprises. (It’s 50 years old!)
Perhaps it’s a good time to reconsider, as we’re past the “NoSQL is cool” hype phase and back to “just use Postgres”, or even “SQLite is good enough”.
I want to focus here on how to keep transactions in the code rather than on their technical comp...
![](https://threedots.tech/media/apple-touch-icon.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- nightmare investigation scenario in which ([View Highlight](https://read.readwise.io/read/01j784t1q9benkb5x4k17bqk45))
	- ![](https://threedots.tech/post/database-transactions-in-go/images/3-no-tx_hu7b1578eb8b5bf6a11bb3dfb279bf30dc_145957_1633x1108_resize_q80_h2_lanczos_3.webp) ([View Highlight](https://read.readwise.io/read/01j788d9yprgxrjgkavp7wk2ev))