title:: Unpopular Opinions About Go (highlights)
author:: [[Three Dots Labs]]
full-title:: "Unpopular Opinions About Go"
category:: #articles
url:: https://threedots.tech/episode/unpopular-opinions-about-go/
summary:: Quick takeaways

Simplicity isn’t enough for complex applications - while Go’s syntax is simple, complex applications still need proper design patterns; primitive code easily becomes spaghetti code in large projects.
Reading the standard library isn’t the best way to learn Go - it’s optimized for different goals than typical applications and might be confusing for beginners.
Router libraries are better than the standard HTTP package - libraries like Chi or Echo come with a nice high-level API.
Struct-based configuration is better than the “optional pattern” - structs are easier to document, discover, and maintain than the popular With-options approach.
There’s no one best project structure - starting small and evolving your structure as needed is better than following a dogmatic approach like the unofficial “Go project layout.”
Writing stubs by hand is better than using mocking libraries - manually written stubs are easier to debug and encourage better interfaces than reflection-based mocking libraries.
Code generation is better than reflect - for ORMs or dependency injection, it gives you compile-time checks and better performance.
Generics are mostly useful for libraries, not application code - while everyone waited for them, they’re rarely needed in typical service-level code.
Channels and goroutines can be overused - they add complexity and should only be used when concurrency is actually needed, not as a default approach.
Go’s error handling is fine for most projects - explicit checks make code easier to read, though built-in stack traces would be helpful.
Memory optimizations are often premature - micro-optimizations waste time for typical API services where network latency is the bottleneck.

Introduction
In this episode of No Silver Bullet, we share some of our unpopular takes on the Go programming language.
After working with Go for eight years on all kinds of projects, we’ve seen many discussions about what idiomatic Go means.
We talk about what worked ...
![](https://threedots.tech/images/favicon_hu0d95454e65643b4b3a38fde8f31a0ef0_20324_144x0_resize_lanczos_3.png)

- Highlights first synced by [[Readwise]] [[May 11th, 2025]]
	- **Struct-based configuration is better than the “optional pattern”** - structs are easier to document, discover, and maintain than the popular With-options approach. ([View Highlight](https://read.readwise.io/read/01jtz0b9vm3pdjj78pxfjycym2))