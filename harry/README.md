# Harry 个人学习工作区 (Codex)

欢迎来到 Harry 针对 **OpenAI Codex** 项目的独立学习与源码研读工作区。

---

## 1. 空间定位与初衷

- **空间用途**：本分支（`harry`）与 [harry/](file:///Users/zhanghongze/PycharmProjects/codex/harry) 独立目录是 Harry 专属的源码学习、系统架构剖析、动态调试实验与知识沉淀空间。
- **上游隔离原则**：
  - 本分支所有内容仅供个人技术研究使用。
  - **严禁向 upstream 主仓库（`openai/codex`）发起 Pull Request**。
  - 不受理任何针对本学习分支的外界 Issue 或 PR。
  - 与上游保持同步时，通过 `git fetch upstream` 及衍合保持主干独立，确保个人学习文档与产出物（图谱、笔记、探究脚本、本地配置）不污染上游工程。

---

## 2. 目录文件导航

| 文件/目录 | 定位说明 | 核心价值 |
| :--- | :--- | :--- |
| [harry/README.md](file:///Users/zhanghongze/PycharmProjects/codex/harry/README.md) | 本工作区总览说明文档 | 明确工作区定位、隔离原则与文档索引导航 |
| [harry/CODEGRAPH.md](file:///Users/zhanghongze/PycharmProjects/codex/harry/CODEGRAPH.md) | 核心架构图谱与符号索引 | 覆盖核心分层、启动流程、关节点输入输出样例与精准符号锚定 |
| [harry/AGENTS.md](file:///Users/zhanghongze/PycharmProjects/codex/harry/AGENTS.md) | 知识型文档与 Agent 行为规范 | 深入剖析系统运转核心机制、关键设计模式与多代理验证机制 |
| [harry/CLAUDE.md](file:///Users/zhanghongze/PycharmProjects/codex/harry/CLAUDE.md) | Agent 极简引导规范 | 为各类 AI 助手进入项目提供统一的阅读指引与上下文契约 |
| [harry/.codegraph/](file:///Users/zhanghongze/PycharmProjects/codex/harry/.codegraph) | 代码索引隔离目录 | 存放本地 CodeGraph 图谱缓存及运行期临时数据（已忽略跟踪） |

---

## 3. Codex 项目全貌概览

OpenAI Codex 是一个基于 Rust 构建的高性能终端智能体（Agent）系统与底层开发套件：

1. **Rust 核心运行时**（位于 [codex-rs/](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs)）：
   - [codex-rs/cli](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/cli)：CLI 统一启动器，支持交互式 TUI、后台守护进程、子命令分发（`exec`、`review`、`mcp`、`app-server` 等）。
   - [codex-rs/tui](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/tui)：基于 Ratatui 构建的全功能终端人机交互界面。
   - [codex-rs/core](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/core)：智能体调度核心，驱动对话状态机、提示词编排、多代理协作、上下文预算与历史压缩。
   - [codex-rs/app-server](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/app-server)：基于 JSON-RPC 2.0 的应用服务器，为 IDE 插件及外接客户端提供统一接口。
   - [codex-rs/sandboxing](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/sandboxing)：跨平台安全沙箱（macOS Seatbelt、Linux Bubblewrap/Landlock、Windows Restricted Token）。
   - [codex-rs/skills](file:///Users/zhanghongze/PycharmProjects/codex/codex-rs/skills)：原生技能发现、解析与动态上下文注入引擎。
2. **多语言 SDK 支持**（位于 [sdk/](file:///Users/zhanghongze/PycharmProjects/codex/sdk)）：
   - [sdk/python](file:///Users/zhanghongze/PycharmProjects/codex/sdk/python)：Python 官方客户端库，覆盖同步与异步交互、工具控制与流式订阅。
   - [sdk/typescript](file:///Users/zhanghongze/PycharmProjects/codex/sdk/typescript)：TypeScript 客户端库，提供结构化输出、线程管理与事件流支持。
