title:: Your Biggest Customer Might Be Your Biggest Bottleneck (highlights)
author:: [[densumesh.dev]]
full-title:: "Your Biggest Customer Might Be Your Biggest Bottleneck"
category:: #articles
url:: https://densumesh.dev/blog/fair-queue/
summary:: A large customer overloaded the system, causing delays for others. Traditional queues serve jobs in order, which lets big jobs block smaller ones. The solution is fair queueing, which shares processing time evenly among all customers.
![](https://densumesh.dev/og-image.png)

- Highlights first synced by [[Readwise]] [[Sep 22nd, 2025]]
	- Traditional queues are like having one massive line at the DMV. Everyone gets served in the order they arrived, which sounds fair until someone shows up with paperwork for 100 vehicles.
	  
	  Fair queueing is different. It’s like having separate lines for each customer, but with a receptionist who calls one person from each line in rotation.
	  
	  ![](https://cdn.densumesh.dev/fair-queue.png)
	  
	  Fair queueing ensures every customer gets a turn, preventing starvation.
	  
	  Busy customers stay busy, but they can’t starve quiet customers. If Customer A has 1,000 messages and Customer B has 1, Customer B doesn’t have to wait for all 1,000 of Customer A’s messages to clear. The system simply alternates between them: A, B, A, A, A… Everyone makes progress. ([View Highlight](https://read.readwise.io/read/01k5pjqm31bncn3gpnyw75j9v3))
		- 💡: 这在golang里太容易实现了。for select。通过对select稍微封装一下，就能获得一个chan的非阻塞读取。