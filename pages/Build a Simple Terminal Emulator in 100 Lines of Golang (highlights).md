title:: Build a Simple Terminal Emulator in 100 Lines of Golang (highlights)
author:: [[Ishuah]]
full-title:: "Build a Simple Terminal Emulator in 100 Lines of Golang"
category:: #articles
url:: https://ishuah.com/2021/03/10/build-a-terminal-emulator-in-100-lines-of-go/
![](https://ishuah.com/images/27.jpg)

- Highlights first synced by [[Readwise]] [[Nov 26th, 2023]]
	- The next step involves connecting to the TTY driver that lives in the kernel. We’ll be using the [pseudoterminal](https://en.wikipedia.org/wiki/Pseudoterminal) for this task. Like the TTY driver, the pseudoterminal lives in the OS kernel. It consists of a pair of pseudo-devices, a pty master, and a pty slave.
	  
	  ![](https://ishuah.com/images/pseudo-terminal.jpg)
	  
	  The pty master and pty slave communicate with each other through the TTY driver
	  
	  The pty slave receives all its input from the pty master. It also sends all its output to pty master. The pty master sends keystrokes from the keyboard to the pty slave. It also prints output from the pty slave to the display. I’ll use [pty](https://github.com/creack/pty), a Go package for interfacing with Unix pseudoterminals. ([View Highlight](https://read.readwise.io/read/01hg3n1155a944nqwe5f40c2tt))
		- **Note**: 1. How does the TTY Subsystem work and what is its current state?
		  2. What is the pseudoterminal and how does it work in communication with the TTY driver?
		  3. How does the program capture and handle keyboard input, and what improvements are made to displaying the output on the screen?
		  
		  Kernel devices meet,
		  Master and slave converse freely,
		  Go package connects.