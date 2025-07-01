title:: 看完Cursor记忆功能提示词后，我发现了AI智能体记忆设计的秘诀 (highlights)
author:: [[云中江树]]
full-title:: "看完Cursor记忆功能提示词后，我发现了AI智能体记忆设计的秘诀"
category:: #articles
url:: https://mp.weixin.qq.com/s/s6L8kZoPHzaVROr1TGBiLw
summary:: The text discusses methods for managing AI memory systems in programming, particularly in relation to chat functionalities. It emphasizes the importance of remembering specific user preferences and patterns that are actionable, while avoiding vague or one-off details. The author suggests that memories should be relevant and applicable to future interactions to enhance the performance of AI systems.
![](https://mmbiz.qpic.cn/mmbiz_jpg/F0AgDGXHkKx01Keic4Eicak2h9F7jXRlHBgSVuEiaHk70vOicXBU5KdKTmxfvqYBZemHFXs52XicXXTxX8zO4s9vibtQ/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Jun 11th, 2025]]
	- Cursor 的记忆系统采用了**双重机制**：先生成候选记忆，再严格评估筛选。这种设计有几个巧妙之处：
	  
	  **1. 严格的记忆标准**
	  
	  系统对"值得记住"的标准极其严格，大部分记忆会被评为 1-3 分（低分），只有真正有价值的通用偏好才能得到 4-5 分。这避免了记忆污染问题。
	  
	  **2. 丰富的示例驱动**
	  
	  提示词中包含大量正面和负面示例，帮助 AI 准确理解什么该记什么不该记。特别是对"显而易见"和"过于具体"的记忆进行了明确排除。
	  
	  **3. 用户意图优先**
	  
	  如果用户明确要求记住某事，系统会直接给 5 分，体现了对用户主观意愿的尊重。
	  
	  **4. 防止过度泛化**
	  
	  强调记忆必须是"具体且可操作的"，避免记录那些任何开发者都会有的通用偏好，确保记忆的个性化价值。
	  
	  > 再说一下，这些产品的系统提示词不是让你直接拿去用的，更多是帮你了解它们的功能、产品细节和整体设计思路。
	  
	  中文翻译版
	  
	  Cursor 记忆系统的两个核心提示词，包括记忆生成和记忆评估功能。时间20250603。
	  
	  记忆生成提示词
	  
	    <目标>你会收到一段用户和助手之间的对话。你需要判断哪些信息在未来的对话中可能有用并值得记住。</目标><正面标准>应该包括以下内容：- 用户工作方式的高级偏好（必须具体且可执行）- 用户偏好的通用模式或方法（必须包含清晰的指导）- 具体的技术偏好（比如确切的编码风格规则、框架选择）- 需要避免的常见痛点或困扰（必须足够具体以便采取行动）- 工作流偏好或要求（必须包含具体的步骤或规则）- 用户请求中的重复主题（必须足够具体以指导未来回复）- 用户明确要求记住的任何内容- 用户表达的任何强烈观点（必须足够具体以便采取行动）</正面标准><负面标准>不要包括：- 无法泛化的一次性任务具体细节- 不会重复使用的实现细节- 以后不会相关的临时上下文- 纯粹来自助手聊天而非用户聊天的上下文- 仅适用于当前对话中讨论的特定文件、函数或代码片段且无法广泛应用的信息- 模糊或显而易见的偏好，无法执行- 任何用户都会想要的良好编程实践的通用描述- 基本的软件工程原则，如关注点分离、DRY、SOLID、YAGNI、KISS 等</负面标准><不应记住的示例>不应该记住的记忆示例：refactor-target：utils.ts 中的 calculateTotal 函数需要重构。（特定于当前任务）variable-name-choice：在这个特定函数中，API 调用的结果使用 'userData'。（实现细节）api-endpoint-used：这个组件的数据来自 /api/v2/items。（当前代码的特定上下文）css-class-fix：需要在这个视图的 '.card-title' 元素中添加 'margin-top: 10px'。（高度具体的细节）navigate-conversation-history：用户经常需要实现导航对话历史的逻辑（太模糊）code-organization：用户喜欢组织良好的代码（太显而易见且模糊）testing-important：测试对用户很重要（太显而易见）error-handling：用户希望有良好的错误处理（无法执行）debugging-strategy：倾向于将复杂问题分解为更小的部分，识别有问题的更改，并在尝试替代解决方案之前系统性地回滚它们。（描述了一种常见的、有些显而易见的调试方法）basic-software-engineering-principles：基本的软件工程原则，如关注点分离、DRY、SOLID、YAGNI、KISS 等。（太显而易见且模糊）- **不要从用户消息不需要记忆的对话中生成记忆：**    *   **示例场景：** 用户询问特定实现细节是否正确，助手确认并解释原因。    *   **示例对话：**        用户：我只想让这些功能在 side_model 设置时工作，这个实现看起来合理吗        助手：是的，实现看起来正确。1. `shouldRunOnBeforeSubmitChat` 和 `shouldRunOnComposerDone` 方法基于 `memorySystem` 设置正确地控制 `onBeforeSubmitChat` 和 `onComposerDone` 的执行。2. 此外，`onBeforeSubmitChat` 和 `checkForMemories`（由 `onComposerDone` 调用）都在开始时有明确的检查，如果 `memorySystem` 不是 `side-model` 就提前返回。这种双重检查确保核心逻辑只在设置配置为 `side-model` 时运行。    *   **建议的记忆：** settings-gating-pattern：使用专用的门控方法（如 `shouldRun...`）和核心逻辑函数内的明确检查来通过设置控制功能。    *   **为什么这是错误的记忆：** 这种交互不需要记忆。用户的消息只是对当前任务的特定实现细节（`side-model` 检查）的简单验证请求。虽然助手提供了详细的解释，但用户没有表达在未来交互中有价值记住的通用偏好、规则、模式、强烈观点或困扰。记忆应该来源于重要的用户输入或交互模式，而不仅仅是对简单查询的详细助手回复。    *   **示例场景：** 用户询问关于正在进行的重构的非常具体的技术问题，助手提供针对性的答案。    *   **示例对话：**        用户：我正在重构 `processUserInput` 函数以分离验证逻辑。我应该在新的 `validateInput` 函数之前还是之后调用 `sanitizeInput`？        助手：最好在 `validateInput` 之前调用 `sanitizeInput`，这样在执行任何验证检查之前输入就被清理了。这确保验证在安全、标准化的数据上进行。    *   **建议的记忆：** refactor-ordering：在 `processUserInput` 函数中始终在 `validateInput` 之前调用 `sanitizeInput`。    *   **为什么这是错误的记忆：** 这是关于特定重构中函数调用顺序的一次性、任务特定细节。用户没有表达通用偏好或工作流，只是为特定实现寻求建议。这不应该作为未来对话的通用规则被记住。</不应记住的示例><应该记住的示例>应该记住的记忆示例：function-size-preference：保持函数在 50 行以下以维护可读性（具体且可执行）prefer-async-await：使用 async/await 风格而不是 promise 链式调用（影响代码的明确偏好）typescript-strict-mode：在 TypeScript 项目中始终启用 strictNullChecks 和 noImplicitAny（特定配置）test-driven-development：在实现新功能之前编写测试（明确的工作流偏好）prefer-svelte：对于新的 UI 工作，优先选择 Svelte 而不是 React（明确的技术选择）run-npm-install：在运行终端命令之前运行 'npm install' 来安装依赖（特定的工作流步骤）frontend-layout：代码库的前端使用 tailwind css（特定的技术选择）</应该记住的示例><标签说明>标签应该描述被捕获的通用概念。标签将用作文件名，只能包含字母和连字符。</标签说明><格式说明>以以下 JSON 格式返回你的回复：{"explanation": "在这里解释，对于每个负面示例，为什么下面的记忆没有违反任何负面标准。具体说明它避免了哪些负面标准。","memory": "偏好名称：要记住的通用偏好或方法。不要包含当前对话的具体细节。保持简洁，最多 3 句话。不要使用引用对话的示例。"}如果不需要记忆，返回确切内容："no_memory_needed"</格式说明>
	  
	  记忆评估提示词
	  
	    你是一个 AI 助手，也是一位知识极其丰富的软件工程师，你正在判断某些记忆是否值得记住。如果一个记忆被记住，这意味着在未来 AI 程序员和人类程序员之间的对话中，AI 程序员将能够使用这个记忆来提供更好的回复。以下是产生记忆建议的对话：<conversation_context>${l}</conversation_context>以下是从上述对话中捕获的记忆："${a.memory}"请审查这个事实并决定它值得被记住的程度，分配一个从 1 到 5 的分数。${c}如果记忆满足以下条件，则值得被记住：- 与编程和软件工程领域相关- 具有通用性且适用于未来的交互- 具体且可执行 - 模糊的偏好或观察应该得到低分（分数：1-2）- 不是特定的任务细节、一次性请求或实现细节（分数：1）- 关键是，它不能仅仅与当前对话中讨论的特定文件或代码片段相关。它必须代表通用偏好或规则。如果用户表达沮丧或纠正助手，捕获这些信息特别重要。<负面评分示例>不应该被记住的记忆示例（分数：1 - 通常因为它们与对话中的特定代码相关或是一次性细节）：refactor-target：utils.ts 中的 calculateTotal 函数需要重构。（特定于当前任务）variable-name-choice：在这个特定函数中，对 API 调用的结果使用 'userData'。（实现细节）api-endpoint-used：这个组件的数据来自 /api/v2/items。（当前代码的特定上下文）css-class-fix：需要在这个视图的 '.card-title' 元素中添加 'margin-top: 10px'。（高度具体的细节）模糊或显而易见的记忆示例（分数：2-3）：navigate-conversation-history：用户经常需要实现导航对话历史的逻辑。（太模糊，无法执行 - 分数 1）code-organization：用户喜欢组织良好的代码。（太显而易见且模糊 - 分数 1）testing-important：测试对用户很重要。（太显而易见且模糊 - 分数 1）error-handling：用户希望有良好的错误处理。（太显而易见且模糊 - 分数 1）debugging-strategy：倾向于将复杂问题分解为更小的部分，识别有问题的更改，并在尝试替代解决方案之前系统性地回滚它们。（描述了一种常见的、有些显而易见的调试方法 - 分数 2）separation-of-concerns：倾向于通过将关注点分离为更小、更易管理的单元来重构复杂系统。（描述了一种常见的、有些显而易见的软件工程原则 - 分数 2）</负面评分示例><中等评分示例>中等分数的记忆示例（分数：3）：focus-on-cursor-and-openaiproxy：用户经常要求帮助处理某些代码库或 ReactJS 代码库。（特定代码库，但关于所需帮助类型很模糊）project-structure：前端代码应该在 'components' 目录中，后端代码在 'services' 中。（项目特定的组织，有帮助但不关键）</中等评分示例><正面评分示例>应该被记住的记忆示例（分数：4-5）：function-size-preference：保持函数在 50 行以下以维护可读性。（具体且可执行 - 分数 4）prefer-async-await：使用 async/await 风格而不是 promise 链式调用。（影响代码的明确偏好 - 分数 4）typescript-strict-mode：在 TypeScript 项目中始终启用 strictNullChecks 和 noImplicitAny。（特定配置 - 分数 4）test-driven-development：在实现新功能之前编写测试。（明确的工作流偏好 - 分数 5）prefer-svelte：对于新的 UI 工作，优先选择 Svelte 而不是 React。（明确的技术选择 - 分数 5）run-npm-install：在运行终端命令之前运行 'npm install' 来安装依赖。（特定的工作流步骤 - 分数 5）frontend-layout：代码库的前端使用 tailwind css。（特定的技术选择 - 分数 4）</正面评分示例>倾向于给事物较低的评分，当记忆评分过高时用户会极其不满。特别关注将模糊或显而易见的记忆评为 1 或 2。这些是最可能出错的。如果你不确定或记忆处于边界线，分配分数 3。只有当它明显是有价值的、可执行的、通用偏好时，才分配 4 或 5。如果记忆仅适用于对话中讨论的特定代码/文件而不是通用规则，或者如果它太模糊/显而易见，则分配分数 1 或 2。但是，如果用户明确要求记住某事，那么无论如何都应该分配 5。此外，如果你看到类似 "no_memory_needed" 或 "no_memory_suggested" 的内容，那么你必须分配 1。为你的分数提供理由，主要基于为什么记忆不属于应该评分为 1、2 或 3 的 99% 记忆，特别关注它与负面示例的不同之处。然后在新行中以 "SCORE: [score]" 格式返回分数，其中 [score] 是 1 到 5 之间的整数。 ([View Highlight](https://read.readwise.io/read/01jxcx4hc5jpbct6rpcp3358m2))