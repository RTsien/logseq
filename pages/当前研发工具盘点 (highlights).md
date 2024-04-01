title:: 当前研发工具盘点 (highlights)
author:: [[woa.com]]
full-title:: "当前研发工具盘点"
category:: #articles
url:: https://iwiki.woa.com/p/4009917234
summary:: The text discusses various tools and processes used to improve development efficiency, such as backend portals, environment tracking, and development documentation. It emphasizes the importance of understanding and utilizing tools like configuration generators, lint tools, and test coverage for efficient coding and bug prevention. Additionally, it highlights the significance of testing phases like unit testing, TDD, and end-to-end testing to ensure code quality and reduce bugs in the development process.
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Mar 30th, 2024]]
	- 我们对代码要求有比较合适的单测覆盖，为了解决测试过程中的一些基础依赖如redis/mongo等，我们提供了`make test-prelude` 指令，它会将你的依赖识别并启动相关容器，你可以像正式运行一样的调试。 ([View Highlight](https://read.readwise.io/read/01ht50jr8vmgwym62ezzhekn19))
		- 💡: 学一下自测过程中存储端的处理方式 #TODO
	- 我们已经在尝试测试（用例）左移，在转测前我们确保自己看过测试同学的用例，据说能有效降低最后的bug量。 我听闻最高效的改bug是研发阶段，越往后成本越高，你觉得呢？ ([View Highlight](https://read.readwise.io/read/01ht50nbtns3f835fr0ekhqxjd))
	- TDD是由测试同学提供的一组业务逻辑测试用例，它不是你可能误解的Test-Drive-Develop，至少目前它们很可能滞后于你的开发，所以驱动不了你。但是我们测试同学不断在新增&优化用例，日常冒烟也会跑到，一些主要流程的异常，相信有了它，我们感知会更早。 ([View Highlight](https://read.readwise.io/read/01ht50pvg6jyzxb1b3r7ba3zdp))
	- e2e是一个测试的补充，它学名端到端测试。在RED后台，主要是弥补关键路径上的单测覆盖不了的情况，有点像系统集成测试。目前会自动化跑的是登录、邮件、路由能力等。它的书写成本较高，常用于稳定且重要的流程中。 ([View Highlight](https://read.readwise.io/read/01ht50pqwxz5rczvrhby4ew6nh))