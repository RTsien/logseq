title:: Finding Needles in a Haystack With Best-of-K (highlights)
author:: [[Marc Brooker's Blog]]
full-title:: "Finding Needles in a Haystack With Best-of-K"
category:: #articles
url:: http://brooker.co.za/blog/2024/03/25/needles.html
summary:: The text discusses the use of the best-of-k algorithm for load balancing in distributed systems, particularly in scenarios where there are capacity limits for each worker. The best-of-k algorithm involves picking k out of n workers and sending requests to the least loaded worker among those k. The text explores an iterative variant of the algorithm to address rejection issues when workers are at capacity. It also delves into the implications of different values of k and c on the algorithm's performance, highlighting that best-of-k is not a magic solution and may require a different approach for scenarios where specific workers with available capacity need to be targeted. Additionally, the text provides a probability distribution calculation for the iterative best-of-k algorithm.
![](https://brooker.co.za:443/blog/favicon.ico)

- Highlights first synced by [[Readwise]] [[Apr 24th, 2024]]
	- Enter best-of-*k*. In this simple algorithm, we pick *k* of the *n* workers, and send the request to the least loaded of those *k*. Typically, *k* is small, like 2 or 3. Best-of-*k* leads to a much better load distribution than random, is much more robust to stale data than best-of-*n*, and can be run in O(1) time. It’s a great pick for a stateless load-balancing algorithm. ([View Highlight](https://read.readwise.io/read/01hw61r7dsxb2s3rc8mdnbjhhg))