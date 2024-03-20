title:: TKEx 容器中运行 eBPF 工具的填坑指南 (highlights)
author:: [[woa.com]]
full-title:: "TKEx 容器中运行 eBPF 工具的填坑指南"
category:: #articles
url:: https://km.woa.com/articles/show/583912?kmref=search&from_page=1&no=8
summary:: TKEx容器平台在Serverless节点托管模式下引入了新的困难与挑战，尤其是在业务应用的性能分析和故障排查方面。使用eBPF技术可以有效改善这种情况，但在TKEx容器中应用eBPF技术时会遇到一些问题。这些问题包括内核头文件依赖缺失或版本不匹配，文件系统debugfs挂载缺失，以及过滤条件不生效等。文章提供了解决这些问题的具体步骤，包括手工安装内核头文件、手工挂载debugfs以及修改工具以便在容器中执行带有过滤条件的工具。文章还提到了一键安装脚本，帮助解决这些问题并适配TKEx容器。
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Mar 17th, 2024]]
	- 每个 TKEx Pod 对应一个轻量虚拟机，虚拟机的内核版本当前统一为 5.4.119-1-tlinux4-0009-eks，满足 eBPF 对内核版本的要求（推荐 4.9 及以上），但是注意后缀 eks，这是由 TKEx（EKS）定制的内核版本。
	  
	  问题是，yum 源中并没有 5.4.119-1-tlinux4-0009-eks 对应版本的内核头文件安装包；且业务容器的基础镜像中已有的内核头文件跟实际运行的内核版本很可能不匹配。总之，坑比较多，没有很好的适配 eBPF 运行环境，不少想用 bcc-tools 的研发小伙伴到这里基本选择放弃了。
	  
	  该如何填坑呢？可通过手工安装 5.4.119-1-tlinux4-0009 版本（移除 eks 后缀）的内核头文件 RPM 包统一解决，具体步骤如下：
	  
	    wget http://mirrors.tencent.com/os/tlinux/Tlinux-Kernel-RPMs/5.4.119-1-tlinux4-0009/kernel-tlinux4-devel-5.4.119-1.0009.tl3.x86_64.rpm
	    
	    rpm -ivh --force kernel-tlinux4-devel-5.4.119-1.0009.tl3.x86_64.rpm
	  
	  手工安装内核头文件还不够，上述执行工具 execsnoop 时报错中还包含了以下信息： ([View Highlight](https://read.readwise.io/read/01hs5k7m8cr9f5c5g3vp4jkwv9))
	- 找不到目录 /lib/modules/5.4.119-1-tlinux4-0009-eks。具体原理不再赘述。执行以下 hack 操作：
	  
	    eks_kernel_version=$(uname -r)
	    installed_kernel_version=${eks_kernel_version%-eks}
	    
	    mkdir -p /lib/modules/$eks_kernel_version
	    ln -s /usr/src/kernels/$installed_kernel_version /lib/modules/$eks_kernel_version/build
	  
	  至此，bcc-tools 依赖的内核头文件已准备就绪 ([View Highlight](https://read.readwise.io/read/01hs5k890eveztnnwq0z4as14c))
	- 该文件属于 tracefs，用于增加或删除 kprobe 探针。TKEx 容器中默认未挂载 tracefs，仅包含了空文件夹 /sys/kernel/debug，该目录一般用于挂载 debugfs。另外，当挂载 debugfs 时，tracefs 会自动挂载在子目录 tracing 上。
	  
	  接下来我们手工挂载 debugfs，具体命令如下：
	  
	    mount -t debugfs debugfs /sys/kernel/debug
	  
	  **问题三：找不到相应的内核函数导致 kprobe 创建失败**
	  
	  继续尝试执行工具 execsnoop，提示找不到 sys_execve 这样的 kprobe：
	  
	  ![](https://km.woa.com/asset/6c029d4b437c416d9a9781e014fb3860?height=376&width=1732)
	  
	  实际上 kprobe 要作用于内核函数 __x64_sys_execve，自然找不到 sys_execve。修改 bcc Python 依赖库中的文件 /usr/lib/python3.6/site-packages/bcc/__init__.py，将 _syscall_prefixes 数组的前两项 b"sys_" 与 __x64_sys_ 调换次序，具体如下：
	  
	    _syscall_prefixes = [
	        b"__x64_sys_",
	        b"sys_",
	        b"__x32_compat_sys_",
	        b"__ia32_compat_sys_",
	        b"__arm64_sys_",
	        b"__s390x_sys_",
	        b"__s390_sys_",
	    ]
	  
	  执行工具 execsnoop，终于可以正常输出了。 ([View Highlight](https://read.readwise.io/read/01hs5k8vs1b63ymm9mws0yn5wp))
	- 具体原因是 bpf 运行于内核态，通过帮助函数 bpf_get_current_pid_tgid() 获取的为全局范围（Host PID Namespace）的 PID，而 PID 181 位于容器的 PID Namespace。除非容器共享 Host PID Namespace，否则判断条件 if (pid != 181) 总是返回 true，导致没有任何输出。
	  
	    int syscall__trace_entry_openat(struct ([View Highlight](https://read.readwise.io/read/01hs5karjw3vy2jc44qjfasjtd))
	- 此，在容器中执行带有过滤条件（-p/-t）的工具，需要修改下工具，将容器 PID 转换为全局 PID。以下是具体的转换函数 get_container_current_tgid()，以 bpf_get_current_pid_tgid() 获取到的全局 PID 作为参数，供参考。
	  
	    BPF_HASH(pidmap, u32, ([View Highlight](https://read.readwise.io/read/01hs5kbw6pv093cven5mmvqyxx))