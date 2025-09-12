title:: 熔断器的设计 (highlights)
author:: [[sentinelguard.io]]
full-title:: "熔断器的设计"
category:: #articles
url:: https://sentinelguard.io/zh-cn/docs/golang/circuit-breaking.html
summary:: The text explains the design of circuit breakers in distributed systems to ensure high availability by preventing cascading failures. Circuit breakers have three states: Closed, Open, and Half-Open, and support strategies like SlowRequestRatio and ErrorRatio for triggering circuit breaks based on response times and error rates. Rules are defined to set thresholds for triggering circuit breaks based on response times, error ratios, or error counts.
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Sep 13th, 2025]]
	- 熔断器模型
	  
	  Sentinel 熔断降级基于熔断器模式 (circuit breaker pattern) 实现。熔断器内部维护了一个熔断器的状态机，状态机的转换关系如下图所示：
	  
	  ![](https://user-images.githubusercontent.com/9434884/82635455-ca075f00-9c32-11ea-9e99-d67518923e0d.png)
	  
	  熔断器有三种状态：
	  
	  1.  Closed 状态：也是初始状态，该状态下，熔断器会保持闭合，对资源的访问直接通过熔断器的检查。
	  2.  Open 状态：断开状态，熔断器处于开启状态，对资源的访问会被切断。
	  3.  Half-Open 状态：半开状态，该状态下除了探测流量，其余对资源的访问也会被切断。探测流量指熔断器处于半开状态时，会周期性的允许一定数目的探测请求通过，如果探测请求能够正常的返回，代表探测成功，此时熔断器会重置状态到 Closed 状态，结束熔断；如果探测失败，则回滚到 Open 状态。
	  
	  这三种状态之间的转换关系这里做一个更加清晰的解释：
	  
	  1.  初始状态下，熔断器处于 Closed 状态。如果基于熔断器的统计数据表明当前资源触发了设定的阈值，那么熔断器会切换状态到 Open 状态；
	  2.  Open 状态即代表熔断状态，所有请求都会直接被拒绝。熔断器规则中会配置一个熔断超时重试的时间，经过熔断超时重试时长后熔断器会将状态置为 Half-Open 状态，从而进行探测机制；
	  3.  处于 Half-Open 状态的熔断器会周期性去做探测。 ([View Highlight](https://read.readwise.io/read/01k4zjgfydngvxvn97rnx8j8jz))
	- 熔断策略
	  
	  Sentinel 熔断器的三种熔断策略都支持静默期 (规则中通过MinRequestAmount字段表示)。静默期是指一个最小的静默请求数，在一个统计周期内，如果对资源的请求数小于设置的静默数，那么熔断器将不会基于其统计值去更改熔断器的状态。静默期的设计理由也很简单，举个例子，假设在一个统计周期刚刚开始时候，第 1 个请求碰巧是个慢请求，这个时候这个时候的慢调用比例就会是 100%，很明显是不合理，所以存在一定的巧合性。所以静默期提高了熔断器的精准性以及降低误判可能性。
	  
	  Sentinel 支持以下几种熔断策略：
	  
	  •   慢调用比例策略 (SlowRequestRatio)：Sentinel 的熔断器不在静默期，并且慢调用的比例大于设置的阈值，则接下来的熔断周期内对资源的访问会自动地被熔断。该策略下需要设置允许的调用 RT 临界值（即最大的响应时间），对该资源访问的响应时间大于该阈值则统计为慢调用。
	  •   错误比例策略 (ErrorRatio)：Sentinel 的熔断器不在静默期，并且在统计周期内资源请求访问异常的比例大于设定的阈值，则接下来的熔断周期内对资源的访问会自动地被熔断。
	  •   错误计数策略 (ErrorCount)：Sentinel 的熔断器不在静默期，并且在统计周期内资源请求访问异常数大于设定的阈值，则接下来的熔断周期内对资源的访问会自动地被熔断。
	  
	  注意：这里的错误比例熔断和错误计数熔断指的业务返回错误的比例或则计数。也就是说，如果规则指定熔断器策略采用错误比例或则错误计数，那么为了统计错误比例或错误计数，需要调用API： `api.TraceError(entry, err)` 埋点每个请求的业务异常。 ([View Highlight](https://read.readwise.io/read/01k4zjkvg1waxb28d2qjg58zp9))
	- 最佳场景实践
	  
	  熔断器一般用于应用对外部资源访问时的保护措施。这里简单描述一些场景：
	  
	  •   分布式系统中降级：假设存在应用A需要调用应用B的接口(特别是一些对接外部公司或者业务的接口时候)，那么一般用于A调用B的接口时的防护；
	  •   数据库慢调用的防护： 假设应用需要读/写数据库，但是该读写SQL存在潜在慢SQL的可能性，那么可以对该读写接口做防护，当接口不稳定时候(存在慢SQL)，那么基于熔断器做降级。
	  •   也可以是应用中任意弱依赖接口做降级防护（即自动降级后不影响业务核心链路）。
	  
	  Example
	  
	  熔断降级 demo 可以参考 [circuit breaker example](https://github.com/alibaba/sentinel-golang/tree/master/example/circuitbreaker) ([View Highlight](https://read.readwise.io/read/01k4zjmcmp7rm55v6e683qg7qn)) #[[card]]