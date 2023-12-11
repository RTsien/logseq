title:: JSON Web Token 入门教程 (highlights)
author:: [[作者： 阮一峰]]
full-title:: "JSON Web Token 入门教程"
category:: #articles
url:: https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html
![](https://www.wangbase.com/blogimg/asset/201807/bg2018072301.jpg)

- Highlights first synced by [[Readwise]] [[Dec 5th, 2023]]
	- JWT 的原理是，服务器认证以后，生成一个 JSON 对象，发回给用户，就像下面这样。
	  
	  >     
	  >     {
	  >       "姓名": "张三",
	  >       "角色": "管理员",
	  >       "到期时间": "2018年7月1日0点0分"
	  >     }
	  >     
	  
	  以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名（详见后文）。
	  
	  服务器就不保存任何 session 数据了，也就是说，服务器变成无状态了，从而比较容易实现扩展。 ([View Highlight](https://read.readwise.io/read/01hgws660s4qqwmv2bmyw80cjr))
	- Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。
	  
	  > •   iss (issuer)：签发人
	  > •   exp (expiration time)：过期时间
	  > •   sub (subject)：主题
	  > •   aud (audience)：受众
	  > •   nbf (Not Before)：生效时间
	  > •   iat (Issued At)：签发时间
	  > •   jti (JWT ID)：编号
	  
	  除了官方字段，你还可以在这个部分定义私有字段，下面就是一个例子。
	  
	  >     
	  >     {
	  >       "sub": "1234567890",
	  >       "name": "John Doe",
	  >       "admin": true
	  >     }
	  >     
	  
	  注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。
	  
	  这个 JSON 对象也要使用 Base64URL 算法转成字符串。 ([View Highlight](https://read.readwise.io/read/01hgwshh39tmywntjjsddhsfz2))
	- Signature 部分是对前两部分的签名，防止数据篡改。
	  
	  首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。
	  
	  >     
	  >     HMACSHA256(
	  >       base64UrlEncode(header) + "." +
	  >       base64UrlEncode(payload),
	  >       secret)
	  >     
	  
	  算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户。 ([View Highlight](https://read.readwise.io/read/01hgwskqnsrxwddsmh3v4e8k4s))
		- 💡: 密钥用于签名，防篡改
	- Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。
	  
	  JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符`+`、`/`和`=`，在 URL 里面有特殊含义，所以要被替换掉：`=`被省略、`+`替换成`-`，`/`替换成`_` 。这就是 Base64URL 算法。 ([View Highlight](https://read.readwise.io/read/01hgwte7as8833fyatsbxjr32h))
	- 客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。
	  
	  此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息`Authorization`字段里面。
	  
	  >     
	  >     Authorization: Bearer <token>
	  >     
	  
	  另一种做法是，跨域的时候，JWT 就放在 POST 请求的数据体里面。 ([View Highlight](https://read.readwise.io/read/01hgwtdamt616v20whmwzmea5k))
	- （4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
	  
	  （5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
	  
	  （6）为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。 ([View Highlight](https://read.readwise.io/read/01hgwtfr2xkjswdrxch3j0228s))