title:: When You Shouldn’t Use Frameworks in Go (highlights)
author:: [[Three Dots Labs]]
full-title:: "When You Shouldn’t Use Frameworks in Go"
category:: #articles
url:: https://threedots.tech/episode/when-you-should-not-use-frameworks/
summary:: Quick takeaways

Frameworks promise productivity but often lead to issues as projects get larger and more complex.
The Go community prefers small, focused libraries over frameworks due to Go’s design philosophy influenced by Unix principles.
Watch out for risks using frameworks like vendor lock-in, deprecation, and costly migrations that can take months.
Explicit code is more maintainable than magic framework abstractions.
Choose your approach based on project size and maturity - frameworks might work for prototypes, while modular libraries are better for long-term projects.

Introduction
In this episode of the No Silver Bullet podcast, we discuss frameworks in Go and when they’re useful or problematic.
We talk about why the Go community generally avoids frameworks compared to other languages, and how small, modular libraries are often preferred in Go development.
We share our experiences with frameworks across different projects, including tradeoffs between productivity and long-term maintenance.
Notes

Model-View-Controller (MVC): Pattern first described in the 1970s for Smalltalk, still widely used today.
Unix Philosophy: design concept created by Ken Thompson (also a Go creator) promoting small programs that do one thing well and work together.
When to avoid DRY in Go
Watermill: Our event-driven library for Go designed to not be a framework.
Repository Pattern: Our blog post that is still relevant and frequently referenced.
tdl and pq - the CLI tools we mentioned.
Clean Architecture: The topic of our next podcast episode, a design approach that helps maintain separation of concerns.
Wild Workouts : Our example Go codebase demonstrating clean architecture.
The Best Go framework: no framework?

Quotes

The happy path is easy enough, but the happy path is usually not the hard part of software. We often overvalue how much effort the boilerplate requires.

Miłosz



Framework knowledge tends to become out of date. You can spend days or weeks learning something abo...
![](https://threedots.tech/images/favicon_hu0d95454e65643b4b3a38fde8f31a0ef0_20324_144x0_resize_lanczos_3.png)

- Highlights first synced by [[Readwise]] [[Mar 23rd, 2025]]
	- **Choose your approach based on project size and maturity** - frameworks might work for prototypes, while modular libraries are better for long-term projects. ([View Highlight](https://read.readwise.io/read/01jptqgk7zq6ds1cze6m768eqk))