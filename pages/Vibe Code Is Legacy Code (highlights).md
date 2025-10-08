title:: Vibe Code Is Legacy Code (highlights)
author:: [[Steve Krouse]]
full-title:: "Vibe Code Is Legacy Code"
category:: #articles
url:: https://blog.val.town/vibe-code
summary:: Vibe coding means using AI to write code quickly without fully understanding it. This works well for small or throwaway projects but creates hard-to-fix legacy code for serious apps. To build lasting software, humans must carefully learn and manage the code, not just rely on AI.
![](https://blog.val.town/og-image.png?title=Vibe+code+is+legacy+code)

- Highlights first synced by [[Readwise]] [[Sep 22nd, 2025]]
	- Despite [widespread](https://simonwillison.net/2025/Mar/19/vibe-coding/) [confusion](https://simonwillison.net/2025/May/1/not-vibe-coding/), Andrej Karpathy [coined "vibe coding"](https://x.com/karpathy/status/1886192184808149383?lang=en) as a kind of AI-assisted coding where you **"forget that the code even exists."**
	  
	  ![](https://pbs.twimg.com/profile_images/1296667294148382721/9Pr6XrPB.jpg)
	  
	  [Andrej Karpathy](https://twitter.com/karpathy) [@karpathy](https://twitter.com/karpathy)
	  
	  [](https://twitter.com/karpathy/status/1903870973126045712)
	  
	  Good post! It will take some time to settle on definitions. Personally I use "vibe coding" when I feel like this dog. My iOS app last night being a good example. But I find that in practice I rarely go full out vibe coding, and more often I still look at the code, I add complexity slowly and I try to learn over time how the pieces work, to ask clarifying questions etc.  
	  
	  Your browser does not support the video tag.
	  
	  [Posted Mar 23, 2025 at 6:06PM](https://twitter.com/karpathy/status/1903870973126045712)
	  
	  Legacy code
	  
	  We already have a phrase for code that nobody understands: **legacy code**.
	  
	  Legacy code is universally despised, and for good reason. But why? You have the code, right? Can't you figure it out from there?
	  
	  Wrong. Code that nobody understands is tech debt. It takes a lot of time to understand unfamiliar code enough to debug it, let alone introduce new features without also introducing bugs. ([View Highlight](https://read.readwise.io/read/01k5pmf6b11vxyfspbez0prntb))
		- 💡: 从定义上来说，纯 vibe coding 获得的代码的确是 legacy code！😄
	- Programming is fundamentally [*theory building*](https://pages.cs.wisc.edu/~remzi/Naur.pdf), not producing lines of code. We know this. This is why we make fun of business people who try to measure developer productivity in lines of code. ([View Highlight](https://read.readwise.io/read/01k5pmk5w4279kaw80z1kdqvav))
		- 💡: 各种的编程范式，系统架构，引入了大量代码。但是核心算法可能就极其简单的 10 行。可是没有一行代码是多余的。
	- Prototypes & throwaway code
	  
	  I've happily vibe coded apps to:
	  
	  I don't needed to continue developing those apps, so it hasn't been a problem that I don't understand their code. These apps are also very small, which means that I haven't incurred that much debt if I need to jump in and read the code at some point. I was able to vibe code these apps way faster than I could've built them, and it was a blast.
	  
	  Vibe coding is a spectrum
	  
	  Vibe coding is on a spectrum of how much you understand the code. The more you understand, the less you are vibing. ([View Highlight](https://read.readwise.io/read/01k5pmrz4rxygrhkxk0vqd16tw))
- New highlights added [[Sep 22nd, 2025]] at 2:37 AM
	- Programming is fundamentally [*theory building*](https://pages.cs.wisc.edu/~remzi/Naur.pdf), not producing lines of code. We know this. This is why we make fun of business people who try to measure developer productivity in lines of code. ([View Highlight](https://read.readwise.io/read/01k5pppzhdpns5tm58m4jq5grb))
		- 💡: 编程思路和代码量并不是线性关系
	- The worst possible situation is to have a non-programmer vibe code a large project that they intend to maintain. This would be the equivalent of giving a credit card to a child without first explaining the concept of debt. ([View Highlight](https://read.readwise.io/read/01k5pptpkkeztwp0yetskyfbkt))
	- If you don't understand the code, your only recourse is to ask AI to fix it for you, which is like paying off credit card debt with another credit card. ([View Highlight](https://read.readwise.io/read/01k5ppxtc4g2c6gwg34rtw0xvf))
	- If you're building something serious that you intend to maintain in 2025, Andrej has the right of it:
	  
	  > [Keep] a very tight leash on this new over-eager junior intern savant with encyclopedic knowledge of software, but who also bullshits you all the time, has an over-abundance of courage and shows little to no taste for good code. And emphasis on being slow, defensive, careful, paranoid, and on always taking the inline learning opportunity, not delegating. ([View Highlight](https://read.readwise.io/read/01k5ppzacb4q22wcnmb6wa0xv6))