title:: 记一次mongodb核心集群雪崩故障分析 (highlights)
author:: [[51cto.com]]
full-title:: "记一次mongodb核心集群雪崩故障分析"
category:: #articles
url:: https://blog.51cto.com/u_14951246/2539931
summary:: The article analyzes a major failure in a MongoDB cluster caused by incorrect client configurations and performance issues, leading to service disruptions. It highlights how sudden spikes in traffic resulted in repeated connection failures, overwhelming system resources and causing a "snowball" effect. The author emphasizes the need for standardized client settings and improvements in the MongoDB kernel to prevent such issues in the future.
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Sep 13th, 2025]]
	- **mongodb内核源码定位分析**  
	  上面的分析已经确定，问题根源是mongodb内核多个线程读取/dev/urandom随机数引起，走读mongodb内核代码，发现读取该文件的地方如下：
	  
	  ![](https://s2.51cto.com/images/blog/202010/04/ced00d873b1c0b7afd86e53cde93cf01.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)
	  
	  上面是生成随机数的核心代码，每次获取随机数都会读取”/dev/urandom”系统文件，所以只要找到使用该接口的地方即可即可分析出问题。  
	  继续走读代码，发现主要在如下地方：  
	  //服务端收到客户端sasl认证的第一个报文后的处理，这里会生成随机数  
	  //如果是mongos，这里就是接收客户端sasl认证的第一个报文的处理流程  
	  Sasl_scramsha1_server_conversation::_firstStep(…) {  
	  … …  
	  unique_ptr<SecureRandom> sr(SecureRandom::create());  
	  binaryNonce[0] = sr->nextInt64();  
	  binaryNonce[1] = sr->nextInt64();  
	  binaryNonce[2] = sr->nextInt64();  
	  … …  
	  }  
	  //mongos相比mongod存储节点就是客户端，mongos作为客户端也需要生成随机数  
	  SaslSCRAMSHA1ClientConversation::_firstStep(…) {  
	  … …  
	  unique_ptr<SecureRandom> sr(SecureRandom::create());  
	  binaryNonce[0] = sr->nextInt64();  
	  binaryNonce[1] = sr->nextInt64();  
	  binaryNonce[2] = sr->nextInt64();  
	  … …  
	  }  
	  **2.5.2 mongodb内核源码随机数优化**  
	  从2.5.1分析可以看出，mongos处理客户端新连接sasl认证过程都会通过"/dev/urandom"生成随机数，从而引起系统sy% CPU过高，我们如何优化随机数算法就是解决本问题的关键。  
	  继续分析mongodb内核源码，发现使用随机数的地方很多，其中有部分随机数通过用户态算法生成，因此我们可以采用同样方法，在用户态生成随机数，用户态随机数生成核心算法如下：  
	  class PseudoRandom {  
	  … …  
	  uint32_t _x;  
	  uint32_t _y;  
	  uint32_t _z;  
	  uint32_t _w;  
	  }
	  
	     该算法可以保证产生的数据随机分布，该算法原理详见：
	     http://en.wikipedia.org/wiki/Xorshift
	     也可以查看如下git地址获取算法实现：
	     mongodb随机数生成算法注释
	    
	       ** 总结：**通过优化sasl认证的随机数生成算法为用户态算法后，CPU sy% 100%的问题得以解决，同时代理性能在短链接场景下有了数倍/数十倍的性能提升。 ([View Highlight](https://read.readwise.io/read/01k4zjrz2zbpsacwpjfkgxm20w))
	- 为什么突发流量业务会抖动？  
	  **答：**由于业务是java业务，采用链接池方式链接mongos代理，当有突发流量的时候，链接池会增加链接数来提升访问mongodb的性能，这时候客户端就会新增链接，由于客户端众多，造成可能瞬间会有大量新连接和mongos建链。链接建立成功后开始做sasl认证，由于认证的第一步需要生成随机数，就需要访问操作系统"/dev/urandom"文件。又因为mongos代理模型是默认一个链接一个线程，所以会造成瞬间多个线程访问该文件，进而引起内核态sy%负载过高。
	  
	  为何mongos代理引起“雪崩”，流量为何跌零不可用？  
	  **答：**原因客户端某一时刻可能因为流量突然有增加，链接池中链接数不够用，于是增加和mongos代理的链接，由于是老集群，代理还是默认的一个链接一个线程模型，这样瞬间就会有大量链接，每个链接建立成功后，就开始sasl认证，认证的第一步服务端需要产生随机数，mongos服务端通过读取"/dev/urandom"获取随机数，由于多个线程同时读取该文件触发内核态spinlock锁CPU sy% 100%问题。由于sy%系统负载过高，由于客户端超时时间设置过小，进一步引起客户端访问超时，超时后重连，重连后又进入sasl认证，又加剧了读取"/dev/urandom"文件，如此反复循环持续。  
	  此外，第一次业务抖动后，服务端扩容了8个mongos代理，但是客户端没有修改，造成B机房业务配置的2个代理在同一台服务器，无法利用mongo java sdk的自动剔除负载高节点这一策略，所以最终造成”雪崩” ([View Highlight](https://read.readwise.io/read/01k4zjv5eab89bbv8qmcb4nf37))
	- 为什么数据节点没有任何慢日志，但是代理负载却CPU sy% 100%？  
	  **答：**由于客户端java程序直接访问的是mongos代理，所以大量链接只发生在客户端和mongos之间，同时由于客户端超时时间设置太短(有接口设置位几十ms，有的接口设置位一百多ms，有的接口设置位500ms)，就造成在流量峰值的时候引起连锁反应(突发流量系统负载高引起客户端快速超时，超时后快速重连，进一步引起超时，无限死循环)。Mongos和mongod之间也是链接池模型，但是mongos作为客户端访问mongod存储节点的超时很长，默认都是秒级别，所以不会引起反复超时建链断链。 ([View Highlight](https://read.readwise.io/read/01k4zjwtk6z3s18r1mak084rbp))
	- **PHP短链接业务，如何规避踩坑**  
	  由于PHP业务属于短链接业务，如果流量很高，不可避免的要频繁建链断链，也就会走sasl认证流程，最终多线程频繁读取"/dev/urandom"文件，很容易引起前面的问题。这种情况，可以采用4.1 java客户端类似的规范，同时不要使用低版本的Linux内核，采用3.x以上内核版本，就可以规避该问题的存在。 ([View Highlight](https://read.readwise.io/read/01k4zjzeqh0jdstbacpvg39dx2))
	- Mongodb内核源码设计与实现分析  
	  本文相关的Mongodb线程模型及随机数算法实现相关源码分析如下：  
	  [https://github.com/y123456yz/reading-and-annotate-mongodb-3.6](https://github.com/y123456yz/reading-and-annotate-mongodb-3.6) ([View Highlight](https://read.readwise.io/read/01k4zjzsvxw1x7zr15gfwg3pxa))