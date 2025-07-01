title:: 为什么大模型(llm)可以控制手机 (highlights)
author:: [[hanzilu.com]]
full-title:: "为什么大模型(llm)可以控制手机"
category:: #articles
url:: http://hanzilu.com/wordpress/?p=376
summary:: Droidrun is an open-source project that allows users to control their Android phones using natural language commands through a large language model (LLM). The process involves a cycle of reasoning and acting, where the ReAct Agent interprets user goals and executes actions based on the LLM's instructions. It uses ADB commands to interact with the phone's UI and retrieves element coordinates through a custom portal app.
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Jun 11th, 2025]]
	- droidrun如何获取UI元素的屏幕坐标
	  
	  上边说了adb命令只能通过屏幕坐标去操作手机，现在用户输入“点击登录按钮”，droidrun是如何获取登录按钮的屏幕坐标的呢？
	  
	  它开发了一个代理程序（com.droidrun.portal）安装到手机上，通过adb命令发送广播和portal app通信，比如adb shell am broadcast -a com.droidrun.portal.GET_ELEMENTS，这个命令就可以获取当前页面中可点击的元素及其属性，属性中就含有元素的坐标值。实际返回结果是这样
	  
	  > [{‘text’: ‘登录’, ‘className’: ‘TextView’, ‘index’: 1, ‘bounds’: ‘39,302,199,501’, ‘resourceId’: ”, ‘type’: ‘clickable’, ‘isParent’: True}] ([View Highlight](https://read.readwise.io/read/01jxd5t09399fcgn7a9brbtra8))
	- ![](https://imgproxy.readwise.io/?url=http%3A//hanzilu.com/wordpress/wp-content/uploads/2025/05/image1.webp&hash=a6ec9c0a1f11a0aca7f45d8107414451) ([View Highlight](https://read.readwise.io/read/01jxd5snsckky0kf2n60qqw0p3))