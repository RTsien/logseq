title:: 案例演示: 𝐆𝐢𝐭 𝐖𝐨𝐫𝐤𝐭𝐫𝐞𝐞 + 𝐂𝐥𝐚𝐮𝐝𝐞 𝐂𝐨𝐝𝐞 同时开发多个功能 (highlights)
author:: [[BadUncle]]
full-title:: "案例演示: 𝐆𝐢𝐭 𝐖𝐨𝐫𝐤𝐭𝐫𝐞𝐞 + 𝐂𝐥𝐚𝐮𝐝𝐞 𝐂𝐨𝐝𝐞 同时开发多个功能"
category:: #tweets
url:: https://x.com/BadUncleX/status/1942289427533029789/?rw_tt_thread=True
summary:: Git worktree lets you create multiple folders in one project, each with its own branch for separate features. Using Claude Code, you can develop these features at the same time without conflicts. This method improves efficiency by allowing parallel work and easy merging later.
![](https://pbs.twimg.com/profile_images/1747922230389231616/yoWxjZFD.jpg)

- Highlights first synced by [[Readwise]] [[Jul 25th, 2025]]
	- Git worktree 让你在同一仓库创建多个工作目录，每个目录独立切换分支。结合 Claude Code，实现真正的并发开发——多个 AI 助手同时工作，互不干扰。
	  
	  实战步骤
	  
	  假设你有 `music_shop` 项目，要并发开发 drums、bases、keyboards 三个功能：
	  
	  1. 创建 worktrees 管理目录  
	  mkdir ../music_shop-worktrees
	  
	  2. 为每个功能创建独立 worktree  
	  \- git worktree add ../music_shop-worktrees/drums -b drums  
	  \- git worktree add ../music_shop-worktrees/bases -b bases  
	  \- git worktree add ../music_shop-worktrees/keyboards -b keyboards
	  
	  上述命令完成三件事：  
	  \- 创建新分支（如 drums）  
	  \- 复制完整代码到新目录  
	  \- 将该目录绑定到对应分支
	  
	  并发开发
	  
	  打开三个终端/编辑器窗口：
	  
	  终端1：开发鼓组功能  
	  cd ../music_shop-worktrees/drums  
	  > "实现鼓组音色选择功能"
	  
	  终端2：开发贝斯功能  
	  cd ../music_shop-worktrees/bases  
	  > "添加贝斯音轨编辑器"
	  
	  终端3：开发键盘功能  
	  cd ../music_shop-worktrees/keyboards  
	  > "创建虚拟键盘界面"
	  
	  关键点：三个 Claude Code 实例并行工作，各自在独立分支上修改代码，无冲突。
	  
	  合并成果
	  
	  回到主项目目录  
	  cd ../music_shop
	  
	  合并所有功能  
	  \- git checkout main  
	  \- git merge drums  
	  \- git merge bases  
	  \- git merge keyboards ([View Highlight](https://read.readwise.io/read/01k0yk3s7pc88kr891p3stfnz5))