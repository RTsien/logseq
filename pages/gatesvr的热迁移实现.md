# GateSvr 优雅关闭与客户端连接迁移机制

本文档详细分析gatesvr的优雅关闭流程和客户端连接迁移机制。
- iwiki: https://iwiki.woa.com/p/4015719243
- ## 优雅关闭流程
- ### 关闭流程图
  
  ```mermaid
  
  graph TD
  
    A[Shutdown Signal Received] --> B[Hook 1: Shard & Request Drain]
  
    B --> C{Check Active Shards & Processing Count}
  
    C --> D["activeShards = region.ActiveShards()"]
  
    D --> E["processingCount = gate.ProcessingCount()"]
  
    E --> F{"activeShards == 0 && processingCount == 0?"}
  
    F -->|No| G[Log: Active shards + remaining requests]
  
    G --> H[Sleep 5 seconds]
  
    H --> C
  
    F -->|Yes| I[Log: Shard cleared, exit in 5s]
  
    I --> J[Sleep 5 seconds - Route table update time]
  
    J --> K[Hook 1 Complete]
  
    K --> L[Hook 2: Component Shutdown Sequence]
  
    L --> M["1. connector.Stop()"]
  
    M --> N[Close stopCh, Wait for WaitGroup]
  
    N --> O["2. reporter.Stop()"]
  
    O --> P[Close stopCh, Wait for WaitGroup]
  
    P --> Q["3. gateHandler.Stop()"]
  
    Q --> R[Stop multicaster]
  
    R --> S[Wait for WaitGroup]
  
    S --> T[Stop goExecutor]
  
    T --> U[Stop ssExecutor]
  
    U --> V[Stop frameCache]
  
    V --> W["4. grpcServer.GracefulStop()"]
  
    W --> X[Log: grpc stop]
  
    X --> Y[Shutdown Complete]
  
    style A fill:#ff9999
  
    style Y fill:#99ff99
  
    style F fill:#ffcc99
  
    style L fill:#ccccff
  
  ```
- ### 详细关闭过程
- #### 阶段1: 请求排空 (Hook 1)
  
  **代码位置**: `cmd/gatesvr/main.go:230-244`
  
  1. **持续监控循环**:
	- 检查 `region.ActiveShards()` - 正在服务的活跃分片
	- 检查 `gate.ProcessingCount()` - 正在处理的请求计数
	  
	  2. **排空条件**:
	- 等待直到 `activeShards == 0` 且 `processingCount == 0`
	- 如果未就绪，记录状态并休眠5秒后重试
	  
	  3. **最终宽限期**:
	- 排空完成后，额外休眠5秒用于路由表更新
	- 确保新请求被正确重定向到其他Pod
- #### 阶段2: 组件关闭 (Hook 2)
  
  **代码位置**: `cmd/gatesvr/main.go:246-255`
  
  **关闭顺序** (关键 - 业务层优先，然后传输层):
  
  1. **Connector** (`connector.go:78-83`):
	- 设置 `stopped = true`
	- 关闭 `stopCh` 通道
	- 通过 `WaitGroup` 等待所有goroutine
	  
	  2. **Reporter** (`reporter.go:144-148`):
	- 关闭 `stopCh` 通道
	- 通过 `WaitGroup` 等待所有goroutine
	  
	  3. **Gate Handler** (`gate_handler.go:211-218`):
	- 停止multicaster (消息广播)
	- 等待handler的 `WaitGroup`
	- 停止执行器池 (goExecutor, ssExecutor)
	- 停止frame cache
	  
	  4. **gRPC Server**:
	- 调用 `grpcServer.GracefulStop()` - 允许现有RPC完成
	- 记录完成日志
- ### 关键设计原则
- #### 1. **请求计数**
- `IncProcessingCount()` / `DecProcessingCount()` 跟踪活跃请求
- 使用位置: `gate_service_server.go:576-577`, `gate_handler.go:2005-2017`
- 确保关闭期间不会丢弃请求
- #### 2. **分片迁移**
- 活跃分片在关闭前迁移到其他Pod
- `region.ActiveShards()` 返回仍在服务的分片
- 防止数据丢失并确保服务连续性
- #### 3. **优雅组件关闭**
- 每个组件都有自己的 `Stop()` 方法
- 使用通道 (`stopCh`) 发送关闭信号
- 使用 `WaitGroup` 确保goroutine干净终止
- #### 4. **关闭任务队列**
- 分片可以排队在关闭后运行的任务: `AddWaitingShutdownTask()`
- 位置: `shard/shard.go:189-194`
- 确保清理任务正确执行
- ## 客户端连接迁移机制
- ### 迁移流程图
  
  ```mermaid
  
  sequenceDiagram
  
    participant C as 客户端
  
    participant OG as 老Gate (Origin)
  
    participant NG as 新Gate (New)
  
    participant R as Region协调器
  
    participant Backend as 后端服务
  
    Note over R: Pod关闭前，Region协调器通知迁移
  
    
  
    R->>NG: AssignShard通知
  
    Note over NG: 创建Shard，状态=StateAssignee
  
    
  
    R->>OG: RemoveShard通知
  
    Note over OG: Shard状态改为StateShutdownDoing
  
    
  
    rect rgb(255, 240, 240)
  
        Note over OG,NG: P2P迁移阶段
  
        NG->>OG: requestHandoff(grpc调用)
  
        Note over OG: DNS检查，确认老Pod可访问
  
        
  
        OG->>OG: OnHandoff() - 收集session信息
  
        Note over OG: 收集MigrateSession数据:<br/>- SessionInfo<br/>- ServerSeq<br/>- MaxClientSeq<br/>- State
  
        
  
        OG->>NG: 返回MigrateSession列表
  
        Note over NG: OnHostShard() - 重建sessions
  
        
  
        loop 每个MigrateSession
  
            NG->>NG: 创建新Session
  
            NG->>NG: 恢复序列号状态
  
        end
  
        
  
        NG->>OG: 迁移完成确认
  
        OG->>OG: OnHandoffEnd()
  
        Note over OG: Shard状态=StateShutdownDone
  
    end
  
    
  
    rect rgb(240, 255, 240)
  
        Note over C,Backend: 客户端重定向阶段
  
        
  
        C->>OG: 新请求到达
  
        Note over OG: Shard状态检查
  
        
  
        alt Shard状态=StateShutdownDoing
  
            OG->>OG: 加入等待shutdown队列
  
            Note over OG: 等待处理完成后重定向
  
        else Shard状态=StateShutdownDone
  
            OG->>C: 立即返回重定向错误
  
            Note over C: 错误码: ErrRedirect<br/>Meta: redirect_to=新地址<br/>Meta: redirect_wait_ms=随机等待
  
        end
  
        
  
        Note over C: 客户端等待后重连到新Gate
  
        C->>NG: 重连请求 (带redirect=1标记)
  
        
  
        alt Session存在且可重定向
  
            NG->>NG: checkRedirectableSession()
  
            NG->>NG: 继承老Session数据
  
            NG->>C: 重定向成功，继续服务
  
            NG->>Backend: 正常转发请求
  
        else Session不存在/不可重定向
  
            NG->>C: 返回ErrLoginAgain
  
            Note over C: 客户端需要重新登录
  
        end
  
    end
  
  ```
- ### 核心迁移机制详解
- #### 1. **Shard状态管理**
  
  **代码位置**: `internal/pkg/gate/shard/shard.go:23-27`
  
  ```go
  
  StateAssignee      *// 初始状态，还未host*
  
  StateNormal        *// 正常服务状态  *
  
  StateShutdownDoing *// 正在迁移中*
  
  StateShutdownDone  *// 迁移完成*
  
  ```
- #### 2. **P2P迁移过程**
  
  **代码位置**: `internal/pkg/gate/shard/region.go:275-400`
  
  1. **DNS检查**: 新Gate先检查老Gate的Pod是否可访问
  
  2. **会话收集**: 老Gate的`OnHandoff()`收集所有活跃session
  
  3. **状态传输**: 包含`SessionInfo`, `ServerSeq`, `MaxClientSeq`, `State`
  
  4. **会话重建**: 新Gate的`OnHostShard()`重建所有session
  
  5. **状态恢复**: 恢复序列号和连接状态
- #### 3. **客户端重定向策略**
  
  **代码位置**: `internal/pkg/gate/gate_handler.go:568-590`, `1544-1620`
- ##### 重定向时机判断:
  
  ```go
  
  *// 迁移准备中 - 等待处理完成*
  
  if errors.Is(err, shard.ErrShardHandoffPreparing) {
  
    submitToWaitingShutdownList() *// 加入等待队列*
  
  }
  
  *// 迁移完成 - 立即重定向*
  
  case shard.ErrShardInHandoff, shard.ErrShardHandoffCompleted:
  
    sendRedirectResponse() *// 立即返回重定向*
  
  ```
- ##### 重定向响应格式:
  
  ```go
  
  Meta{
  
    "redirect_to": "新Gate地址",
  
    "redirect_wait_ms": "随机等待时间(0-2000ms)"
  
  }
  
  ```
- #### 4. **Session继承机制**
  
  **代码位置**: `internal/pkg/gate/gate_handler.go:722-740`, `2075-2085`
  
  1. **重定向标记检查**: 请求需要带`redirect=1`标记
  
  2. **Session匹配**: 通过`OpenID`匹配迁移过来的session
  
  3. **状态继承**:
	- GID绑定
	- 序列号状态
	- 自定义数据
	  
	  4. **验证机制**: 确保session可重定向(`IsRedirectable()`)
- ### 关键设计亮点
- #### 1. **零丢包保证**
- 迁移准备期间，请求进入等待队列而非立即重定向
- 确保所有处理中的请求完成后才进行重定向
- #### 2. **渐进式重定向**
- 随机等待时间(`MaxClientRedirectWaitMs=2000`)避免客户端同时重连
- DNS检查确保目标Pod可访问才进行迁移
- #### 3. **状态一致性**
- 完整的序列号状态迁移(`ServerSeq`, `MaxClientSeq`)
- Session状态和自定义数据的完整传输
- #### 4. **容错处理**
- 迁移失败时的回退机制
- 重复连接的检测和处理
- 超时session的自动清理
- ### 性能优化
  
  1. **并发迁移**: 多个shard同时进行P2P迁移
  
  2. **批量传输**: MigrateSession支持批量传输(`maxHandoffSessionCountOnce`)
  
  3. **DNS优化**: 预检查避免无效迁移尝试
  
  4. **指标监控**: `redirect_success_cnt`等指标跟踪迁移效果
- ## 重启考虑
  
  优雅关闭确保了:
- **零停机时间**: 请求在关闭前排空
- **数据一致性**: 分片完全迁移
- **干净状态**: 所有goroutine和资源被清理
- **路由更新**: 为负载均衡器更新路由留出时间
  
  这种设计支持计划内重启(部署)和紧急关闭，同时保持服务可用性。整套机制确保了在Pod关闭/重启过程中，客户端连接能够无缝迁移到新的Gateway实例，实现真正的零停机时间。