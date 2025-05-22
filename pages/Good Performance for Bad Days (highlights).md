title:: Good Performance for Bad Days (highlights)
author:: [[Marc Brooker's Blog]]
full-title:: "Good Performance for Bad Days"
category:: #articles
url:: http://brooker.co.za/blog/2025/05/20/icpe.html
summary:: Marc Brooker emphasizes the need for performance evaluation to focus on how systems behave under overload, not just in ideal conditions. He points out that many studies highlight good performance metrics but overlook critical issues like metastability and real-world workloads. Closing the gap between theory and practice in performance evaluation is essential for system reliability and availability.
![](https://brooker.co.za:443/blog/favicon.ico)

- Highlights first synced by [[Readwise]] [[May 22nd, 2025]]
	- The core of what I tried to communicate is that, in my view, a lot of the performance evaluation community is overly focused on *happy case* performance (throughput, latency, scalability), and not focusing as much as we need to on performance under saturation and overload.
	  
	  ![](http://brooker.co.za/blog/images/icpe_s1.png)
	  
	  In fact, the opposite is potentially more interesting. For builders and operators of large systems, a lack of performance predictability under overload is a big driver of unavailability. ([View Highlight](https://read.readwise.io/read/01jvsss9h608zxd74xqevb0mhy))
	- This is a common theme in postmortems and outage reports across the industry. Overload drives systems into regimes they aren’t used to handling, which leads to downtime. Sometimes, in the case of [metastable failures](https://brooker.co.za/blog/2021/05/24/metastable.html), this leads to downtime that persists even after the overload has passed.
	  
	  How did we get into this situation?
	  
	  *Not Measuring the Hard Stuff*
	  
	  At least one reason is immediately obvious if you pay attention to the performance evaluation in the majority of systems papers. Most of them show throughput, latency, or some other measure of goodness at a load far from the saturation point of the system.
	  
	  ![](http://brooker.co.za/blog/images/icpe_s3.png)
	  
	  The first-order reason for this is unsurprising: folks want to show the system they built in a good light. But there are some second-order reasons too. One is that performance evaluation is easiest, and most repeatable, in this part of the performance curve, and it takes expertise that many don’t have to push beyond it.
	  
	  Some bolder authors will compare saturation points, showing that their systems are able to do more good stuff even when the load is excessive.
	  
	  ![](http://brooker.co.za/blog/images/icpe_s4.png)
	  
	  Only the boldest will go beyond this saturation point to show the performance of their system under truly excessive amounts of load, after the point where performance starts to drop. ([View Highlight](https://read.readwise.io/read/01jvsstqepkaye04pytzszvvfm))
	- This regime is important, because it’s very hard to compose reliable end-to-end systems without knowing where the saturation points of components are, and how they perform beyond that point. Even if you try do things like rate limiting and throttling at the front door, which you should, you still need to know how much you can send, and what the backend looks like when it starts saturating.
	  
	  As a concrete example, TCP uses latency and loss as a signal to slow down, and assumes that if everybody slows down congestion will go away. These nice, clean, properties don’t tend to be true of more complex systems. ([View Highlight](https://read.readwise.io/read/01jvst3vgnqfpxj3vt9ccq26ak))
- New highlights added [[May 22nd, 2025]] at 1:17 AM
	- If you read only one performance-related systems paper in your life, make it [Open Versus Closed: A Cautionary Tale](https://www.usenix.org/legacy/event/nsdi06/tech/full_papers/schroeder/schroeder.pdf). This paper provides a crucial mental model for thinking about the performance of systems. Here’s the key image from that paper:
	  
	  ![](http://brooker.co.za/blog/images/open_closed.png)
	  
	  When we look at the performance space, we see two things:
	  
	  1.  Most cloud systems are *open* (APIs, web sites, web services, MCP servers, whatever)
	  2.  Most benchmarks are *closed* (TPC-C, YCSB, etc)
	  
	  That doesn’t make sense. The most famous downside of this disconnect is [coordinated omission](https://www.scylladb.com/2021/04/22/on-coordinated-omission/), where we massively underestimate the performance impact of tail latency. But that’s far from the whole picture. Closed benchmarks are too kind to realistically reflect how performance changes with load, for the simple reason that they slow their load down when latency goes up. ([View Highlight](https://read.readwise.io/read/01jvst7wb17f3pc07j310wpcyb))
	- *Metastability*
	  
	  As I’ve [written about before](https://brooker.co.za/blog/2021/05/24/metastable.html) ([a few times](https://brooker.co.za/blog/2019/05/01/emergent.html), [in different ways](https://brooker.co.za/blog/2022/06/02/formal.html)), metastability is a problem that distributed systems engineers need to pay more attention to. Not paying attention to performance under overload means not paying attention to metastability, where the majority of real-world triggers are overload-related.
	  
	  Metastability isn’t some esoteric problem. It can be triggered by retries. ([View Highlight](https://read.readwise.io/read/01jvstk753aah5wjq25tak8h7t))
	- The example I used was TPC-C, which has coordination patterns that are much easier to scale than most real-world workloads. [When visualized as a graph of rows that transact together](https://brooker.co.za/blog/2024/02/12/parameters.html), that becomes clear - you can basically partition on *warehouse* and avoid all cross-partition write-write conflicts. ([View Highlight](https://read.readwise.io/read/01jvstgs35amb07pttxh46c1ev))