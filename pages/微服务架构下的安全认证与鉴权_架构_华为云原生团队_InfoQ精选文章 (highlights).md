title:: 微服务架构下的安全认证与鉴权_架构_华为云原生团队_InfoQ精选文章 (highlights)
author:: [[infoq.cn]]
full-title:: "微服务架构下的安全认证与鉴权_架构_华为云原生团队_InfoQ精选文章"
category:: #articles
url:: https://www.infoq.cn/article/uib7ark52myroa2kl7xf
![](https://static001.geekbang.org/static/infoq/img/infoq_icon.jpg)

- Highlights first synced by [[Readwise]] [[Nov 26th, 2023]]
	- 客户端 Token 方案
	  
	  令牌在客户端生成，由身份验证服务进行签名，并且必须包含足够的信息，以便可以在所有微服务中建立用户身份。令牌会附加到每个请求上，为微服务提供用户身份验证，这种解决方案的安全性相对较好，但身份验证注销是一个大问题，缓解这种情况的方法可以使用短期令牌和频繁检查认证服务等。对于客户端令牌的编码方案，Borsos 更喜欢使用 JSON Web Tokens（JWT），它足够简单且库支持程度也比较好。 ([View Highlight](https://read.readwise.io/read/01hg3vcsadzj6dnyh2ny69phwd))
	- 在微服务架构下，每个微服务拆分的粒度会很细，并且不只有用户和微服务打交道，更多还有微服务间的调用。这个时候上述两个方案都无法满足，就要求必须要将 Session 从应用服务器中剥离出来，存放在外部进行集中管理。可以是数据库，也可以是分布式缓存，如 Memchached、Redis 等。这正是 David Borsos 建议的第二种方案，分布式 Session 方案。 ([View Highlight](https://read.readwise.io/read/01hg3vt77a8kfgh96njsvkf0x7))
	- **基于 Token 的认证**
	  
	  
	  
	  随着 Restful API、微服务的兴起，基于 Token 的认证现在已经越来越普遍。Token 和 Session ID 不同，并非只是一个 key。Token 一般会包含用户的相关信息，通过验证 Token 就可以完成身份校验。像 Twitter、微信、QQ、GitHub 等公有服务的 API 都是基于这种方式进行认证的，一些开发框架如 OpenStack、Kubernetes 内部 API 调用也是基于 Token 的认证。基于 Token 认证的一个典型流程如下：
	  
	  
	  
	  •   用户输入登录信息（或者调用 Token 接口，传入用户信息），发送到身份认证服务进行认证（身份认证服务可以和服务端在一起，也可以分离，看微服务拆分情况了）。
	    
	  •   身份验证服务验证登录信息是否正确，返回接口（一般接口中会包含用户基础信息、权限范围、有效时间等信息），客户端存储接口，可以存储在 Session 或者数据库中。
	    
	  •   用户将 Token 放在 HTTP 请求头中，发起相关 API 调用。
	    
	  •   被调用的微服务，验证 Token 权限。
	    
	  •   服务端返回相关资源和数据。
	    
	  
	  
	  
	  基于 Token 认证的好处如下：
	  
	  
	  
	  •   服务端无状态：Token 机制在服务端不需要存储 session 信息，因为 Token 自身包含了所有用户的相关信息。
	    
	  •   性能较好，因为在验证 Token 时不用再去访问数据库或者远程服务进行权限校验，自然可以提升不少性能。
	    
	  •   支持移动设备。
	    
	  •   支持跨程序调用，Cookie 是不允许跨域访问的，而 Token 则不存在这个问题。
	    
	  
	  
	  
	  下面会重点介绍两种基于 Token 的认证方案 JWT/Oauth2.0。 ([View Highlight](https://read.readwise.io/read/01hg3vfw3t6c2vet1f32mkwdqj))
	- JWT 介绍
	  
	  JSON Web Token（JWT）是为了在网络应用环境间传递声明而执行的一种基于 JSON 的开放标准（RFC 7519）。来自 JWT RFC 7519 标准化的摘要说明：JSON Web Token 是一种紧凑的，URL 安全的方式，表示要在双方之间传输的声明。JWT 一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该 Token 也可直接被用于认证，也可被加密。
	  
	  
	  
	  **JWT 认证流程**
	  
	  
	  
	  •   客户端调用登录接口（或者获取 token 接口），传入用户名密码。
	    
	  •   服务端请求身份认证中心，确认用户名密码正确。
	    
	  •   服务端创建 JWT，返回给客户端。
	    
	  •   客户端拿到 JWT，进行存储（可以存储在缓存中，也可以存储在数据库中，如果是浏览器，可以存储在 Cookie 中）在后续请求中，在 HTTP 请求头中加上 JWT。
	    
	  •   服务端校验 JWT，校验通过后，返回相关资源和数据。 ([View Highlight](https://read.readwise.io/read/01hg3vhhmt7rjrj19hafnnj8qd))
	- OAuth 2.0 的流程如下：
	  
	  
	  
	  ![](https://static001.infoq.cn/resource/image/a6/b5/a6d1c5945cd180b384bca7836ced06b5.png)
	  
	  
	  
	  （A）用户打开客户端以后，客户端要求用户给予授权。（B）用户同意给予客户端授权。（C）客户端使用上一步获得的授权，向认证服务器申请令牌。（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。（E）客户端使用令牌，向资源服务器申请获取资源。（F）资源服务器确认令牌无误，同意向客户端开放资源。 ([View Highlight](https://read.readwise.io/read/01hg3vp453vmh6xsy2n9dfye44))
	- 正如 David Borsos 所建议的一种方案，在微服务架构下，我们更倾向于将 Oauth 和 JWT 结合使用，Oauth 一般用于第三方接入的场景，管理对外的权限，所以比较适合和 API 网关结合，针对于外部的访问进行鉴权（当然，底层 Token 标准采用 JWT 也是可以的）。
	  
	  
	  
	  JWT 更加轻巧，在微服务之间进行访问鉴权已然足够，并且可以避免在流转过程中和身份认证服务打交道。当然，从能力实现角度来说，类似于分布式 Session 在很多场景下也是完全能满足需求，具体怎么去选择鉴权方案，还是要结合实际的需求来。 ([View Highlight](https://read.readwise.io/read/01hg3vqew49vcv5tct8yxc3w8g))