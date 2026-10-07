# Harry 工作区 Agent 规范与系统知识剖析

欢迎来到 Harry 的 Codex 独立学习工作区。这份文档梳理了 Harry 在探索和学习 Codex 项目过程中的核心约定、架构通俗解读，以及智能体协助开发时的自检标准。目标是用大白话把系统搞懂，并且保持代码干净、规矩明确。

---

## 1. 工作区核心约定与边界约束

进入本项目的任何 Agent 必须严格遵守以下红线：

1. **分支基准**：本项目当前工作基准分支为 `harry` 分支。所有学习成果、图谱维护、探究笔记与实验代码仅在 `harry` 分支演进，严禁向 `origin/main` 或上游原始仓库提交任何变更。
2. **目录隔离**：所有学习产生的分析文档、架构图谱、Agent 引导文件、探究脚本以及测试产物，必须严格集中存放在 [harry/](file:///Users/zhanghongze/PycharmProjects/codex/harry) 目录下，绝不污染上游开源仓库原有的目录结构。
3. **上游保护**：`upstream`（`openai/codex`）仅作为代码更新源，绝对禁止向 `upstream` 发起 Pull Request 或推送分支。
4. **符号锚定与代码对照**：所有文档中提及代码位置时，必须使用具体的「文件路径 + 类名/函数名/结构体名」，严禁使用会随代码修改而漂移的物理行号，且必须与实际源码一一对照。

---

## 2. 系统核心概念大白话解析

为了让所有人都能直观理解 Codex 的运转奥秘，这里抛开晦涩的技术黑话，用通俗直白的方式把核心概念讲清楚：

### 2.1 Thread（线程）与 Turn（回合）
- **Thread（聊天本与上下文管家）**：
  在源码中对应 [`codex-rs/core/src/codex_thread.rs::CodexThread`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/core/src/codex_thread.rs)。你可以把它想象成一本持续记录的“工作台账”。只要你在一个项目里跟 Codex 交流，这个台账就一直在。它管着你选了哪个模型、用了什么安全策略、过去聊过了什么、以及什么时候需要把太长的历史记录做精简压缩。
- **Turn（工作回合）**：
  对应 [`codex-rs/app-server-protocol/src/protocol/v2/thread_data.rs::Turn`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/app-server-protocol/src/protocol/v2/thread_data.rs)。这是最核心的工作单元。你向它提一个需求（比如“帮我修一下这个报错”），它从开始阅读代码、思考对策、执行终端命令、改写文件，直到最后把结论答复给你，这一整套完整连贯的动作合起来，就是一个 Turn。

### 2.2 Session（现场施工队长）
- 源码对应 [`codex-rs/core/src/session/session.rs::Session`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/core/src/session/session.rs)。
- 如果说 `CodexThread` 是项目经理，那么 `Session` 就是在一线盯着干活的施工队长。在一个 Turn 里，Session 负责一步一步往前推（Step 循环）：这一步把当前文件夹里有哪些文件、系统时间、你的规则打包告诉大模型；大模型如果说“我要执行命令 `cargo test`”，Session 就接过这个需求，找安全沙箱去跑，跑完把结果再喂给大模型，直到大模型说“活干完了”。

### 2.3 Sandboxing（防爆安全沙箱）
- 源码对应 [`codex-rs/sandboxing/src/manager.rs::SandboxManager`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/sandboxing/src/manager.rs)。
- 智能体执行命令是非常危险的（万一删除了关键系统文件或者偷偷联网泄露数据）。Codex 在这方面下了极深的功夫：任何命令都不会在你的电脑里“裸奔”，而是被扔进专门的防爆箱里跑。
  - 在苹果电脑（macOS）上，它使用系统的 Seatbelt 机制（[`codex-rs/sandboxing/src/seatbelt.rs`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/sandboxing/src/seatbelt.rs)），生成类似防火墙的沙箱配置文件。
  - 在 Linux 上，它使用 Bubblewrap 或 Landlock（[`codex-rs/sandboxing/src/bwrap.rs`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/sandboxing/src/bwrap.rs)）。
  - 沙箱严格限制：只能读写当前项目目录，禁止乱翻用户家目录，禁止未授权的外部网络访问。

### 2.4 Skills（专属技能手册）
- 源码对应 [`codex-rs/skills/src/loading.rs::LoadedSkills`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/skills/src/loading.rs)。
- 就像给智能体准备的操作规程。每个 Skill 包含一个 `SKILL.md`，写清楚什么时候该用这个技能、有哪些步骤。当用户提到相关话题或特定任务时，Skills 模块会自动扫描并解析 frontmatter 头部信息（也就是 Markdown 文件开头用 `---` 夹在中间的属性说明，写着技能名字和触发条件），将相关技能动态挂载到系统提示词中。

### 2.5 Multi-Agent（协同作战的多智能体）
- 源码对应 [`codex-rs/core/src/tools/handlers/multi_agents_v2.rs`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/core/src/tools/handlers/multi_agents_v2.rs)。
- 当一个任务太复杂、涉及面太广时，单打独斗容易思路混乱。Codex 允许主智能体派生出独立的子智能体（Subagent）。主智能体给子智能体明确的子目标（如“去帮我独立验证这份文档是否有AI味”），子智能体独立开会话跑完后向主智能体交差，主智能体再统一汇总把关。

---

## 3. 架构主链路全流程

整个 Codex 的工作链路可以清晰划分为以下四个步骤：

1. **指令接收与参数组装**：
   - 终端用户在 TUI 输入文字，或者外部程序通过 JSON-RPC 发送 `turn/start` 请求（载荷为 [`TurnStartParams`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/app-server-protocol/src/protocol/v2/turn.rs)）。
   - 服务端将其转化为内部提交项，通过 [`CodexThread::submit`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/core/src/codex_thread.rs) 投递进队列。
2. **环境感知与上下文编排**：
   - 调度核心读取当前工作区文件状态（`WorldState`），合并系统自带规则、用户个人提示词（`DeveloperInstructions`）与匹配到的技能（`Skills`）。
   - 将组装完毕的完整上下文转换为模型支持的输入格式。
3. **模型推理与工具决策**：
   - 调用 [`ModelProvider`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/model-provider) 与大模型通信，采用流式传输与事件总线（像打字机一样一个字一个字实时传回来；同时把思考过程和答复文字实时推送到系统内部的消息广播通道里，让前台界面能马上看到动静），持续派发思考过程（`AgentReasoning`）与回复文本（`AgentMessage`）。
   - 一旦模型产出工具调用需求（比如执行命令或修改文件），进入安全审批与沙箱执行阶段。
4. **安全执行与反馈闭环**：
   - 工具调度与编排器 [`ToolOrchestrator`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/core/src/tools/orchestrator.rs) 与 [`ToolRouter`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/core/src/tools/router.rs) 检查审批模式（是否需要人工按回车确认）。
   - 通过 [`SandboxManager`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/sandboxing) 包装命令并在沙箱中落地执行。
   - 命令的标准输出、错误输出或文件修改差异被统一封装，作为下一步输入重新喂回给大模型，开始下一轮推理，直到模型给出最终结果。

---

## 4. 关键模块深度拆解

| 模块 | 关键源文件 | 功能核心说明 |
| :--- | :--- | :--- |
| **启动与分发** | [`codex-rs/cli/src/main.rs`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/cli/src/main.rs) | 项目的统一总开关。负责命令行参数读取、多平台路径适配、守护进程状态检查，以及将控制权转交给 TUI 或其他子命令。 |
| **交互式终端界面** | [`codex-rs/tui/src/lib.rs`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/tui/src/lib.rs) | 终端交互的主战场。基于 Rust 的 Ratatui 库构建，负责把事件流实时渲染在屏幕上，处理键盘输入、历史记录翻页与快捷键触发。 |
| **应用服务层** | [`codex-rs/app-server/src/lib.rs`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/app-server/src/lib.rs) | 解耦界面与核心的桥梁。支持标准输入输出（stdio）、Unix 域套接字以及 WebSocket，让各种 IDE 插件能像搭积木一样调用 Codex 能力。 |
| **协议定义层** | [`codex-rs/protocol/src/protocol.rs`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/protocol/src/protocol.rs) | 内部通用语言。定义了系统所有流转的消息格式（`EventMsg`）和操作指令（`Op`），确保前后端各模块步调一致。 |
| **核心执行引擎** | [`codex-rs/core/src/codex_thread.rs`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/core/src/codex_thread.rs) | 整个项目的心脏。调度会话状态机、控制多轮上下文长度、处理超时、中断和异常恢复。 |
| **沙箱安全守护** | [`codex-rs/sandboxing/src/manager.rs`](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/sandboxing/src/manager.rs) | 系统的防护罩。根据当前操作系统自动选择合适的沙箱机制，把代码改动与终端命令牢牢锁在安全边界内。 |

---

## 5. Agent 自检与子代理验证修复机制

为确保所有沉淀文档的高质量与高可用性，文档编写完成后必须通过专门的子智能体（Subagent）进行验证与迭代修复。

### 5.1 验证标准四项准则
1. **通俗人话**：必须保证文档中的所有语言都是自然人话，通俗易懂，严禁套用虚头巴脑的官话与套话。
2. **拒绝 AI 味**：行文风格真实、真诚、有逻辑，杜绝“综上所述”、“总而言之”、“具有里程碑意义”等陈词滥调。
3. **通识化解释**：禁止“用专业的名词去解释专业的名词”，所有术语必须有生活化比喻或具体代码上下文支撑。
4. **逐条代码对照**：引用的每一个函数名、结构体名、参数名和文件路径，必须在源码中真实存在，严禁任何形式的凭空捏造。

### 5.2 递归修复工作流
- **开箱即验**：主 Agent 在完成阶段产出后，调用 `invoke_subagent` 启动专门的验证子 Agent 进行审核。
- **开新子 Agent 修复**：若审核中发现任何不符合上述准则的地方，主 Agent **必须启动全新的子 Agent**（绝对不复用原有的子 Agent）针对性修复该问题。
- **多轮复核**：修复完成后，再启动新的验证子 Agent 复核，递归修复至少 3 轮，直至全部合格。
