-
- # **MongoDB Go Driver 超时控制与连接池行为深度解析**
- ## **摘要**
  
  本文旨在为使用 Go 语言与 MongoDB 进行开发的工程师提供一份关于操作超时、错误处理及连接池管理的详尽技术报告。报告将深入剖析 Go 语言的 context 包如何与 MongoDB Go Driver 协同工作，以实现对数据库操作的生命周期控制。我们将详细阐述当一个操作因 context 超时而终止时，驱动程序内部发生的确切行为，特别是针对底层 TCP 连接的处理方式。此外，报告还将探讨客户端超时对 MongoDB 服务器端操作的影响，揭示潜在的数据一致性风险，并最终提供一套完整的、用于构建高可用、高性能和强韧性应用的架构原则与实践建议。
  
  ---
- ## **第一章：基础概念：context 的异步控制与连接池的资源管理**
  
  在深入探讨用户查询的核心问题之前，必须首先建立对两个关键技术支柱的清晰理解：Go 语言中用于管理操作生命周期的 context 包，以及 MongoDB 驱动程序中用于管理网络资源的连接池。独立地理解这两个概念，是分析它们之间复杂相互作用的前提。
- ### **1.1 Go 语言中的 context.Context 范式：一种取消与截止时间机制**
  
  Go 语言的 context 包提供了一种在 API 边界之间以及多个 goroutine 之间传递截止时间（deadlines）、取消信号（cancellation signals）和其他请求范围值的标准方法 1。它并非一种通用的参数传递机制，其核心使命是控制和协调并发操作的生命周期。
- #### **1.1.1 使用 context.WithTimeout 实现超时控制**
  
  在 Go 应用中实现对操作的超时控制，最典型和规范的模式是使用 context.WithTimeout 函数。其基本用法如下 3：
  
  Go
  
  ctx, cancel := context.WithTimeout(parentCtx, 5\*time.Second)
  
  这行代码包含几个关键组成部分：
  
  * parentCtx：父上下文。通常，这是一个来自上游（如 HTTP 请求的 r.Context()）或通过 context.Background() 创建的根上下文。子上下文会继承父上下文的所有特性，包括其取消信号。  
  * 5\*time.Second：超时时长。从这一刻起，一个 5 秒的倒计时开始。  
  * ctx：返回的子上下文。这个新的 context 实例携带了这个 5 秒的截止时间。开发者应将此 ctx 传递给所有需要受此超时控制的函数调用，例如 MongoDB 驱动程序的 FindOne 或 InsertOne 方法。  
  * cancel：一个 context.CancelFunc 类型的函数。调用此函数会立即取消该子上下文及其所有派生的孙上下文，无论超时是否到达。
- #### **1.1.2 defer cancel() 的重要性**
  
  在创建带超时的上下文后，几乎总能看到 defer cancel() 这行代码。这不仅是良好的编程习惯，更是防止资源泄漏的关键措施 2。
  
  defer 语句确保了 cancel() 函数在当前函数返回前一定会被调用。这有两个目的：
  
  1. 如果操作在超时期限内成功完成，调用 cancel() 会立即释放与该子上下文关联的计时器和其他内部资源。  
  2. 如果不调用 cancel()，这些资源只有在父上下文被取消或超时到达时才会被释放，这可能导致不必要的内存和计算资源占用，即“上下文泄漏”。
- #### **1.1.3 检测超时错误**
  
  当上下文的截止时间到达后，任何监听该上下文状态的操作都会被通知。上下文的 Err() 方法会返回一个非 nil 的错误。具体来说，当因为超时而取消时，Err() 方法返回的值是 context.DeadlineExceeded 1。
  
  应用程序可以通过 errors.Is(err, context.DeadlineExceeded) 来精确地判断一个操作失败的原因是否是客户端设定的超时 4。这使得应用能够将超时失败与其他类型的失败（如网络错误、认证失败等）区分开来，并采取不同的处理策略。这一机制是后续错误处理策略的基础。
- ### **1.2 MongoDB Go Driver 连接池的架构原理**
  
  许多开发者误认为 mongo.Client 实例代表一个单一的数据库连接。这是一个根本性的误解。实际上，一个 mongo.Client 实例是一个高度复杂的、线程安全的连接池管理器，它为 MongoDB 拓扑中的每个服务器维护一个独立的连接池 5。在整个应用程序的生命周期中，最佳实践是创建一个单一的
  
  mongo.Client 实例并复用它 8。
- #### **1.2.1 连接的生命周期**
  
  连接池中的每一个 TCP 连接都遵循一个明确的生命周期，这个过程对开发者是透明的：
  
  * **创建 (Creation)**：当应用程序的并发操作（goroutines）需要执行数据库命令时，连接池会按需打开新的套接字（socket）连接，直到达到 maxPoolSize 的上限 5。  
  * **检出与归还 (Checkout & Check-in)**：一个操作会从池中“检出”一个可用连接，使用它发送命令和接收响应，然后在操作完成后将其“归还”到池中。通过连接监控事件，我们可以观察到 ConnectionCheckedOut 和 ConnectionCheckedIn 这两个事件，它们分别标志着连接的检出与归还 9。  
  * **空闲与修剪 (Idle & Pruning)**：归还到池中的连接会处于空闲状态，等待被下一个操作复用。maxIdleTimeMS 参数定义了一个空闲连接在被关闭和移出连接池之前可以保持空闲的最长时间。这有助于在保持性能和节约服务器资源之间取得平衡 5。
- #### **1.2.2 关键配置参数**
  
  正确配置连接池对于优化应用程序的性能和韧性至关重要。下表整合了多个官方文档源中的关键配置参数，为后续的调优建议提供了一个实用的参考指南。
  
  一个普遍存在的误解是，连接池是一个静态且脆弱的实体，任何连接的关闭都是一种需要避免的“失败”。然而，连接池的本质是一个动态的、具有自愈能力的系统。ConnectionPoolCreated、ConnectionClosed、ConnectionCreated 等监控事件的存在证明了连接池的设计初衷就是管理连接的整个生命周期，包括创建和关闭 9。因此，单个连接的关闭并非连接池的失败，而是其正常工作的一部分。连接池的职责是有效地管理这些事件，并根据应用程序的负载动态维持所需数量的健康连接。这种理解将我们的关注点从“保护单个连接”转移到“正确调优整个系统”。
  
  **表 1: MongoDB Go Driver 连接池核心配置选项**
  
  | 参数 | options.Client 方法 / URI 选项 | 描述 | 默认值 | 相关文档 |
  | :---- | :---- | :---- | :---- | :---- |
  | maxPoolSize | SetMaxPoolSize | 每个服务器的连接池中允许的最大连接数。当所有连接都在使用中时，新的操作将会等待。 | 100 | 5 |
  | minPoolSize | SetMinPoolSize | 每个服务器的连接池中维持的最小连接数，即使它们处于空闲状态。 | 0 | 5 |
  | maxIdleTimeMS | SetMaxConnIdleTime | 一个连接在被关闭并从池中移除前，可以保持空闲的最长时间（毫秒）。 | 无限制 | 5 |
  | waitQueueTimeoutMS | SetWaitQueueTimeout | 一个 goroutine 等待从池中获取可用连接的最长时间（毫秒）。 | 无限制 | 5 |
  | connectTimeoutMS | SetConnectTimeout | 与服务器建立新 TCP 连接的超时时间（毫秒）。 | 30 秒 | 6 |
  | socketTimeoutMS | SetSocketTimeout | 套接字读/写操作的超时时间（毫秒）。此设置与 context 的截止时间相互作用。 | 无限制 | 6 |
  
  ---
- ## **第二章：超时操作的剖析：深入驱动程序内部**
  
  本章将直接回应用户的核心问题，通过剖析当 context 超时发生时驱动程序内部的行为，来阐明“关闭连接”与“归还到池”之间的确切含义。
- ### **2.1 上下文的旅程：飞行前检查**
  
  在网络数据包被发送之前，MongoDB Go Driver 已经开始检查 context 的状态。官方文档详细描述了这些发生在操作执行前的检查点 10：
  
  * **服务器选择 (Server Selection)**：如果驱动程序无法立即为操作选择一个合适的服务器（例如，在副本集选举期间），它会进入一个循环等待阶段。在每次循环迭代后，驱动程序都会检查 context 是否已过期。如果 context 的截止时间已到，或者等待时间超过了 serverSelectionTimeoutMS 的设置，操作将立即失败并返回一个服务器选择超时错误 10。  
  * **连接检出 (Connection Checkout)**：在成功选择服务器后，驱动程序会尝试从该服务器的连接池中获取一个连接。如果池中所有连接都已被检出（即达到 maxPoolSize），操作将进入等待队列。如果 context 在此等待期间过期，操作将失败并返回一个超时错误，此时甚至还没有获得网络连接 10。
- ### **2.2 飞行中超时：解决“关闭”与“归还”的模糊性**
  https://docs.google.com/document/d/17trRKYTz6sVSCaA2dtnFTTv6rxtMGNwjjUkUOeJNsok/edit?usp=sharing
  本节将聚焦于最关键的场景：当一个操作已经检出了连接，向服务器发送了命令，并在等待响应的过程中（即在套接字读/写期间）发生超时。
- #### **2.2.1 官方文档的明确说明**
  
  MongoDB Go Driver 的官方文档对此有明确且权威的描述 10。驱动程序会启动一个专门的 goroutine 来监听
  
  context 的取消信号。原文指出：“如果该 goroutine 检测到取消，它会**关闭连接** (closes the connection)”。随后，正在阻塞的 Read() 或 Write() 方法会返回一个I/O错误，驱动程序捕获这个错误，并用 context.Canceled 或相关的上下文错误覆盖它，最终返回给应用程序。
- #### **2.2.2 “未知状态”原则与安全性的至高无上** #card
  id:: 6863f962-2231-4c55-8f1b-2f25c113c59a
  
  一个常见的误解，如在某些社区问答中所见 11，认为“关闭连接”仅仅意味着将其标记为空闲并“归还到池中”。这种说法是一种危险的过度简化。
  
  驱动程序之所以选择彻底关闭（即销毁）TCP连接，而不是复用它，是基于一个至关重要的系统安全原则：**在超时发生后，该连接的状态是未知的，并且可能已被破坏**。一份来自 MySQL Go Driver 维护者的解释，虽然针对不同数据库，但揭示了所有健壮的数据库驱动程序普遍遵循的设计哲学 12：当超时发生时，无法确定其根本原因。可能是服务器查询缓慢，也可能是网络中途出现问题。例如，在读取响应期间超时，可能导致只有部分 BSON 数据被读入本地的 TCP 缓冲区。如果这个“被污染”的连接被归还到池中并被另一个 goroutine 复用，那么后续的操作可能会读取到不完整或错误的数据，导致协议反序列化失败（例如，出现 15 中提到的
  
  incomplete read of message header 错误）、数据损坏或完全不可预测的应用程序行为。
  
  因此，唯一安全的选择是销毁这个连接——即关闭底层的套接字——并将其从池中彻底移除。MongoDB Go Driver 的行为完全符合这一“安全优于性能”的设计权衡。用户希望保留连接以追求性能是可以理解的，但这会违反驱动程序提供的基本安全保证。
- ### **2.3 对连接池的影响：韧性的体现**
  
  这一系列的内部行为最终导向一个令用户安心的结论：单个超时操作导致的连接关闭，**并不会影响整个连接池的健康和可用性**。mongo.Client 及其管理的连接池仍然保持完全活跃。
  
  当这个超时的连接被关闭后，池的内部计数器会减一。如果此时有其他操作需要连接，并且池中的连接总数低于 maxPoolSize，连接池会简单地建立一个新的 TCP 连接来满足需求 5。这个过程的“成本”仅仅是一次新的 TCP 握手延迟，而这正是连接池设计用来应对和管理的正常事件。连接监控事件中的
  
  ConnectionClosed 之后，很可能会跟随一个 ConnectionCreated 事件，这直观地证明了连接池对这类事件的恢复能力 9。
  
  ---
- ## **第三章：服务器端的影响与数据一致性**
  
  探讨完客户端驱动程序的行为后，我们必须将视野扩展到分布式系统的另一端：当客户端因超时而“消失”时，MongoDB 服务器上会发生什么？这揭示了一个开发者必须应对的关键异步性问题。
- ### **3.1 服务器的视角：一个断开的客户端**
  
  MongoDB 服务器对客户端的 context 截止时间一无所知。从服务器的角度看，客户端超时最终表现为客户端的 TCP 连接被突然关闭。服务器对此的反应取决于它在连接断开时正在执行的操作类型。
- ### **3.2 读操作 vs. 写操作：两种不同的结局**
  
  * **读操作 (find, aggregate)**：对于长时间运行的查询，当服务器在准备或发送结果时检测到客户端连接已断开，它通常会终止这个正在执行的查询并回收相关资源。服务器知道结果已经没有接收方了。这种行为与社区讨论中的描述一致 11。  
  * **写操作 (insertOne, updateOne, delete)**：这是风险最高的地方。如果服务器在连接断开之前已经完整接收到了写命令，那么它极有可能继续执行并完成该操作。客户端的消失并不会“撤销”一个已经开始执行的写操作。
- ### **3.3 “幽灵写入”现象与幂等性的必要性**
  
  一个真实的案例完美地展示了这种风险 13。在一个测试中，客户端的更新操作因网络问题超时并返回了错误，但服务器端的
  
  $inc（增量更新）操作实际上已经成功执行。客户端在收到错误后可能会进行重试，这导致该字段被错误地增加了两次。
  
  这个现象揭示了一种危险的解耦：客户端对操作状态的认知（因超时而失败）与服务器上操作的实际状态（已成功）发生了分歧。这个三阶效应意味着，任何使用客户端超时机制且执行非幂等写操作的应用程序，都面临着数据损坏的风险。
  
  **幂等性 (Idempotency)** 是指一个操作执行一次和执行多次产生的效果是相同的。因此，一个至关重要的架构建议是：在使用客户端超时模式时，所有可能被重试的写操作都必须被设计成幂等的。
- ### **3.4 与服务器端超时 (maxTimeMS) 的对比**
  
  为了提供一个完整的视图，我们将客户端的 context 超时与服务器端的 maxTimeMS 选项进行比较 14。
  
  maxTimeMS 是一个与命令一起发送到服务器的参数。由服务器自身来强制执行时间限制。当超时发生时，服务器会干净地终止操作，并向客户端返回一个明确的 MaxTimeMSExpired 错误。这种方法避免了“幽灵写入”问题，因为超时是由服务器管理和通信的。其权衡之处在于，它需要在命令级别单独设置超时，可能不如 context 提供的、可在整个应用调用链中传播的超时机制灵活。
  
  ---
- ## **第四章：综合分析与实践建议**
  
  本章将前述所有分析综合成一个明确的答案，并为构建健壮的 Go 应用程序提供可操作的策略。
- ### **4.1 最终答案与规范实现**
- #### **4.1.1 综合性回答**
  
  对于用户的核心问题，最终的、精确的回答是：“是的，您可以使用 context.WithTimeout 来终止一个长时间运行的查询并立即收到错误。然而，出于安全考虑，此行为在设计上会导致该操作所使用的特定 TCP 连接被关闭和丢弃。连接**池**本身保持完全运行，不受影响。驱动程序的这种行为是正确的，它保护了您的应用程序免受使用状态不确定连接的风险。”
- #### **4.1.2 附注代码示例**
  
  以下是一个完整的、可用于生产环境的 Go 代码片段，它演示了所有最佳实践 3：
  
  ```Go
  
  package main
  
  import (  
  "context"  
  "errors"  
  "fmt"  
  "log"  
  "time"
  
  "go.mongodb.org/mongo-driver/bson"  
  "go.mongodb.org/mongo-driver/mongo"  
  "go.mongodb.org/mongo-driver/mongo/options"  
  )
  
  func main() {  
  // 1\. 设置客户端并连接  
  clientOpts := options.Client().ApplyURI("mongodb://localhost:27017")  
  client, err := mongo.Connect(context.TODO(), clientOpts)  
  if err\!= nil {  
  log.Fatalf("Failed to connect to MongoDB: %v", err)  
  }  
  defer func() {  
  if err := client.Disconnect(context.TODO()); err\!= nil {  
  	log.Fatalf("Failed to disconnect from MongoDB: %v", err)  
  }  
  }()
  
  collection := client.Database("testdb").Collection("items")
  
  // 2\. 为查询操作创建一个带超时的上下文  
  // 假设我们期望查询在 2 秒内完成  
  ctx, cancel := context.WithTimeout(context.Background(), 2\*time.Second)  
  // 3\. 使用 defer cancel() 确保资源被释放  
  defer cancel()
  
  // 模拟一个可能耗时较长的查询  
  filter := bson.M{"$where": "function() { sleep(3000); return true; }"}
  
  var result bson.M  
  err \= collection.FindOne(ctx, filter).Decode(\&result)
  
  // 4\. 详细的错误处理  
  if err\!= nil {  
  // 4.1. 精确判断是否为上下文超时错误  
  if errors.Is(err, context.DeadlineExceeded) {  
  	// 这是我们预期的超时情况  
  	fmt.Println("Query timed out as expected (context deadline exceeded).")  
  	// 在这里，可以记录日志、返回特定的API错误码或触发重试逻辑  
  	return  
  }
  
  // 4.2. 处理其他可能的错误  
  if errors.Is(err, mongo.ErrNoDocuments) {  
  	fmt.Println("No documents found for the given filter.")  
  } else {  
  	// 其他未知错误，例如网络问题、BSON解析错误等  
  	log.Printf("An unexpected error occurred: %v\\n", err)  
  }  
  } else {  
  fmt.Printf("Query successful, found document: %v\\n", result)  
  }  
  }
  ```
- ### **4.2 构建高韧性应用的策略**
- #### **4.2.1 错误处理与重试机制**
  
  * **将 context.DeadlineExceeded 视为“状态未知”**：当收到此错误时，应用程序应假定操作的最终状态是不确定的。  
  * **读操作重试**：对于读操作，在短暂的指数退避后重试通常是安全的。  
  * **写操作重试**：对于写操作，只有在操作是幂等的情况下才能安全地重试。实现幂等性的一些方法包括：  
  * 在插入的文档中包含一个由客户端生成的、唯一的事务ID。在重试时，首先检查该ID是否存在。  
  * 使用 MongoDB 的事务功能，并在事务中包含业务逻辑。
- #### **4.2.2 系统的整体调优**
  
  * **应用层超时**：在应用程序的入口点（如 HTTP handler）设置一个总的、积极但现实的超时。这个超时应该成为整个请求处理过程的“主时钟” 2。  
  * **连接池调优**：使用第一章表格中的参数来调优连接池。例如，如果由于网络延迟导致超时频发，可能需要适度增加 maxPoolSize 来应对连接被关闭和重建的流失率，同时设置一个合理的 waitQueueTimeoutMS 来防止在负载高峰期出现无限的请求排队 5。  
  * **服务器端考量**：对于那些至关重要的、长时间运行的操作，如果必须确保服务器端能被终止，可以考虑在客户端 context 的基础上，额外使用 maxTimeMS 选项 14。
  
  ---
- ## **第五章：结论：超时行为的统一模型**
  
  本报告通过深入分析，构建了一个关于 MongoDB Go Driver 超时行为的完整模型。最后，我们对整个过程进行总结，以巩固核心要点。
- ### **5.1 生命周期回顾**
  
  一个带超时的数据库操作的完整生命周期如下：
  
  1. 应用程序使用 context.WithTimeout 创建一个带截止时间的上下文，并发起数据库调用。  
  2. 驱动程序在多个阶段（服务器选择、连接检出）检查上下文的截止时间 10。  
  3. 如果在套接字 I/O 期间发生超时，驱动程序为了保证安全，会主动关闭该 TCP 连接 10。  
  4. 应用程序的阻塞调用返回，并收到一个 context.DeadlineExceeded 错误。  
  5. 具有韧性的连接池会处理这个被关闭的连接，并在需要时创建新连接以维持服务能力 5。
- ### **5.2 核心要点重申**
  
  1. **安全第一**：驱动程序关闭超时连接是为了防止潜在的数据损坏和协议错误，这是一个不可妥协的、正确的设计选择。  
  2. **连接池的韧性**：连接池并非脆弱的。它被设计用来有效管理连接的流失与再生。用户对连接池整体崩溃的担忧是没有根据的。  
  3. **客户端-服务器的异步性**：客户端超时不保证服务器端操作被同步取消。这使得对于写操作而言，实现幂等性成为一项至关重要的架构要求 13。
- ### **5.3 最终陈述**
  
  MongoDB Go Driver 对基于 context 的超时机制的实现是健壮、安全且符合现代分布式系统最佳实践的。成功的关键不在于试图对抗或绕过这种行为，而在于深刻理解它，并在此基础上构建与之和谐共存的应用程序级逻辑（如精细的错误处理、安全的重试策略和幂等性设计）。通过这种方式，开发者可以充分利用 Go 语言和 MongoDB 的强大功能，构建出真正高性能、高可用的后端服务。
- #### **Works cited**
  
  1. Golang Context Deadline Exceeded \- Uptrace, accessed June 30, 2025, [https://uptrace.dev/glossary/context-deadline-exceeded](https://uptrace.dev/glossary/context-deadline-exceeded)  
  2. How to manage database timeouts and cancellations in Go \- Alex Edwards, accessed June 30, 2025, [https://www.alexedwards.net/blog/how-to-manage-database-timeouts-and-cancellations-in-go](https://www.alexedwards.net/blog/how-to-manage-database-timeouts-and-cancellations-in-go)  
  3. mongodb/mongo-go-driver: The Official Golang driver for ... \- GitHub, accessed June 30, 2025, [https://github.com/mongodb/mongo-go-driver](https://github.com/mongodb/mongo-go-driver)  
  4. Handle Context Deadline Exceeded error in Go (Golang) \- GOSAMPLES, accessed June 30, 2025, [https://gosamples.dev/context-deadline-exceeded/](https://gosamples.dev/context-deadline-exceeded/)  
  5. FAQ \- Go Driver v2.2 \- MongoDB Docs, accessed June 30, 2025, [https://www.mongodb.com/docs/drivers/go/current/faq/](https://www.mongodb.com/docs/drivers/go/current/faq/)  
  6. Connection Pool Overview \- Database Manual \- MongoDB Docs, accessed June 30, 2025, [https://www.mongodb.com/docs/manual/administration/connection-pool-overview/](https://www.mongodb.com/docs/manual/administration/connection-pool-overview/)  
  7. Connection Pool Overview \- Database Manual v6.0 \- MongoDB Docs, accessed June 30, 2025, [https://www.mongodb.com/docs/v6.0/administration/connection-pool-overview/](https://www.mongodb.com/docs/v6.0/administration/connection-pool-overview/)  
  8. MongoDB connections to high \- Reddit, accessed June 30, 2025, [https://www.reddit.com/r/mongodb/comments/1c322yx/mongodb\_connections\_to\_high/](https://www.reddit.com/r/mongodb/comments/1c322yx/mongodb_connections_to_high/)  
  9. Connection Monitoring \- Go Driver v2.2 \- MongoDB Docs, accessed June 30, 2025, [https://www.mongodb.com/docs/drivers/go/current/fundamentals/monitoring/connection-monitoring/](https://www.mongodb.com/docs/drivers/go/current/fundamentals/monitoring/connection-monitoring/)  
  10. Context \- Go Driver v2.2 \- MongoDB Docs, accessed June 30, 2025, [https://www.mongodb.com/docs/drivers/go/current/fundamentals/context/](https://www.mongodb.com/docs/drivers/go/current/fundamentals/context/)  
  11. Does cancelling the context for a query using the MongoDB Go ..., accessed June 30, 2025, [https://stackoverflow.com/questions/70779021/does-cancelling-the-context-for-a-query-using-the-mongodb-go-driver-affect-runni](https://stackoverflow.com/questions/70779021/does-cancelling-the-context-for-a-query-using-the-mongodb-go-driver-affect-runni)  
  12. context.DeadlineExceeded error causes network connection close · Issue \#1631 · go-sql-driver/mysql \- GitHub, accessed June 30, 2025, [https://github.com/go-sql-driver/mysql/issues/1631](https://github.com/go-sql-driver/mysql/issues/1631)  
  13. $inc fails on client but MongoDB processes it anyway, accessed June 30, 2025, [https://www.mongodb.com/community/forums/t/inc-fails-on-client-but-mongodb-processes-it-anyway/121419](https://www.mongodb.com/community/forums/t/inc-fails-on-client-but-mongodb-processes-it-anyway/121419)  
  14. Terminate Running Operations \- Database Manual \- MongoDB Docs, accessed June 30, 2025, [https://www.mongodb.com/docs/manual/tutorial/terminate-running-operations/](https://www.mongodb.com/docs/manual/tutorial/terminate-running-operations/)  
  15. Context Deadline Exceeded Issue Cursor All Golang \- Drivers \- MongoDB, accessed June 30, 2025, [https://www.mongodb.com/community/forums/t/context-deadline-exceeded-issue-cursor-all-golang/233361](https://www.mongodb.com/community/forums/t/context-deadline-exceeded-issue-cursor-all-golang/233361)