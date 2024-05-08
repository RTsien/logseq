title:: Ginkgo学习笔记 (highlights)
author:: [[Alex]]
full-title:: "Ginkgo学习笔记"
category:: #articles
url:: https://blog.gmem.cc/ginkgo-study-note
summary:: Ginkgo is a testing framework for Go language that follows Behavior-Driven Development style. It integrates with Gomega library and allows describing program behaviors in a series of "Specs". Tests can be run using the command "ginkgo" or "go test".
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Apr 20th, 2024]]
	- 断言失败
	  
	  除了调用Gomega之外，你还可以调用Fail函数直接断言失败：
	  
	  Go
	  
	  1
	  
	  Fail("Failure reason")
	  
	  Fail会记录当前进行的测试，并且触发panic，当前Spec的后续断言不会再进行。
	  
	  通常情况下Ginkgo会从panic中恢复，并继续下一个测试。但是，如果你启动了一个Goroutine，并在其中触发了断言失败，则不会自动恢复，必须手工调用GinkgoRecover：
	  
	  Go
	  
	  1
	  
	  2
	  
	  3
	  
	  4
	  
	  5
	  
	  6
	  
	  7
	  
	  8
	  
	  9
	  
	  10
	  
	  11
	  
	  It("panics in a goroutine", func(done Done) {
	  
	    go func() {
	  
	        // 如果doSomething返回false则下面的defer会确保从panic中恢复
	  
	        defer GinkgoRecover()
	  
	        // Ω和Expect功能相同
	  
	        Ω(doSomething()).Should(BeTrue())
	  
	        // 在Goroutine中需要关闭done通道
	  
	        close(done)
	  
	    }()
	  
	  }) ([View Highlight](https://read.readwise.io/read/01hvvm07s69p4jy2n0zm8sh2g6))
	- 断言
	  
	  Ω/Expect
	  
	  两种断言语法本质是一样的，只是命名风格有些不同：
	  
	  Go
	  
	  1
	  
	  2
	  
	  3
	  
	  4
	  
	  5
	  
	  6
	  
	  Ω(ACTUAL).Should(Equal(EXPECTED))
	  
	  Expect(ACTUAL).To(Equal(EXPECTED))
	  
	  Ω(ACTUAL).ShouldNot(Equal(EXPECTED))
	  
	  Expect(ACTUAL).NotTo(Equal(EXPECTED))
	  
	  Expect(ACTUAL).ToNot(Equal(EXPECTED)) ([View Highlight](https://read.readwise.io/read/01hvvm3aqtretgxzns61a3cwh6))
	- 错误处理
	  
	  对于返回多个值的函数：
	  
	  Go
	  
	  1
	  
	  2
	  
	  3
	  
	  4
	  
	  5
	  
	  6
	  
	  func DoSomethingHard() (string, error) {}
	  
	  result, err := DoSomethingHard()
	  
	  // 断言没有发生错误
	  
	  Ω(err).ShouldNot(HaveOccurred())
	  
	  Ω(result).Should(Equal("foo"))
	  
	  对于仅仅返回一个error的函数： 
	  
	  Go
	  
	  1
	  
	  2
	  
	  3
	  
	  func DoSomethingHard() (string, error) {}
	  
	  Ω(DoSomethingSimple()).Should(Succeed()) ([View Highlight](https://read.readwise.io/read/01hvvm3s1et7nyx5qstggqkge0))
	- 断言注解
	  
	  进行断言时，可以提供格式化字符串，这样断言失败可以方便的知道原因：
	  
	  Go
	  
	  1
	  
	  2
	  
	  3
	  
	  4
	  
	  5
	  
	  Ω(ACTUAL).Should(Equal(EXPECTED), "My annotation %d", foo)
	  
	  Expect(ACTUAL).To(Equal(EXPECTED), "My annotation %d", foo)
	  
	  Expect(ACTUAL).To(Equal(EXPECTED), func() string { return "My annotation" }) ([View Highlight](https://read.readwise.io/read/01hvvm48t1w2pm7k7hjss2f08x))