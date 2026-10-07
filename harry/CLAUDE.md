# Claude Agent Instructions (Harry Workspace)

本项目已配置 Harry 独立个人学习工作区。所有代码分析、架构索引、学习规范与工作约定请优先查阅：

1. [harry/AGENTS.md](file:///Users/zhanghongze/PycharmProjects/codex/harry/AGENTS.md)：工作区最高规范与核心知识剖析（核心概念解析、运行主链路、关键模块拆解、子代理验证准则）
2. [harry/CODEGRAPH.md](file:///Users/zhanghongze/PycharmProjects/codex/harry/CODEGRAPH.md)：核心架构图谱与精准符号索引（Mermaid 全景图、启动链路、模块符号清单、数据结构样例）
3. [harry/README.md](file:///Users/zhanghongze/PycharmProjects/codex/harry/README.md)：工作区总览、隔离原则与使用说明

### 核心铁律
- **分支基准**：默认在 `harry` 分支工作，不向上游原始仓库（`openai/codex`）提交任何 PR。
- **目录隔离**：所有学习笔记、架构文档、探究脚本与图谱数据一律存放在 `harry/` 目录下。
- **符号锚定**：引用代码坐标一律使用「文件路径 + 函数名/类名」，禁止物理行号。
