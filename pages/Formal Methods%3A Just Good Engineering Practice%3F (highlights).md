title:: Formal Methods: Just Good Engineering Practice? (highlights)
author:: [[Marc Brooker's Blog]]
full-title:: "Formal Methods: Just Good Engineering Practice?"
category:: #articles
url:: http://brooker.co.za/blog/2024/04/17/formal.html
summary:: Formal methods are valuable in software engineering for large-scale and critical systems. They can help reduce rework and improve efficiency during the design and implementation phases. Using tools like TLA+ can lead to faster system development with fewer trade-offs between correctness and performance.
![](https://lobste.rs/apple-touch-icon-144.png)

- Highlights first synced by [[Readwise]] [[Apr 22nd, 2024]]
	- Once an API or system has customers, changing it becomes many times more expensive and difficult. This is fundamentally related to [Hyrum’s Law](https://www.hyrumslaw.com/):
	  
	  > With a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody.
	  
	  Isolating the behavior of systems with APIs is an excellent idea. In fact, I would consider it one of the most important ideas in all of software engineering. Hyrum’s Law reminds us of the limitations of the approach: users will end up depending on every conceivable implementation detail of an API. This doesn’t mean change is impossible. I have been involved in many projects in my career that have completely re-implemented the system behind APIs. It merely means that change is expensive, and that abstractions like APIs don’t effectively change that reality. ([View Highlight](https://read.readwise.io/read/01hw2kw5h5knt4pah47rs962fq))