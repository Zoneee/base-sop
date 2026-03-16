# Base SOP

AI 辅助开发工作流标准操作规程（SOP）— 可执行版本，覆盖从需求收集到破坏性重构的完整生命周期。

## 文档导航

| 文档 | 说明 |
|------|------|
| [docs/ai-workflow-sop.md](./docs/ai-workflow-sop.md) | **主 SOP 文档** — 阶段化流程、模型选择阈值、质量门、团队角色 |
| [docs/prompt-templates.md](./docs/prompt-templates.md) | **Prompt 模板集** — 6 个可直接复制使用的 prompt 模板 |
| [docs/adr-template.md](./docs/adr-template.md) | **ADR 模板** — 架构决策记录模板（含签署流程） |
| [DECISIONS.md](./DECISIONS.md) | **决策日志** — 版本化的重要 AI 决策记录 |
| [DESIGN.md](./DESIGN.md) | **设计文档** — 项目高层架构与接口文档模板 |
| [.github/workflows/ai-generated-pr-check.yml](./.github/workflows/ai-generated-pr-check.yml) | **CI 工作流** — AI 生成 PR 的自动检查（摘要、测试、变更规模门控、破坏性变更检测） |

## 快速开始

1. **阅读** [docs/ai-workflow-sop.md](./docs/ai-workflow-sop.md) 了解完整流程与阈值。
2. **选择** 对应阶段的 [Prompt 模板](./docs/prompt-templates.md) 开始与 AI 协作。
3. **记录** 重要决策到 [DECISIONS.md](./DECISIONS.md)（引用格式：`DECISION:vX.Y`）。
4. **配置** CI 工作流（将 `.github/workflows/ai-generated-pr-check.yml` 中的占位符替换为实际工具）。

## SOP 阶段一览

| 阶段 | 名称 | 推荐模型 |
|------|------|----------|
| 1 | 需求收集（Intake） | Gemini Pro / Claude Sonnet |
| 2 | 高层架构（Architecture） | Claude Opus / GPT-5 Codex Max |
| 3 | 任务拆解（Planning） | GPT-4o / o3-mini |
| 4 | 日常开发（Development） | GPT-4 Turbo / Claude Haiku / Gemini Flash |
| 5 | 需求变更（Replanning） | GPT-4o → 大模型（超出阈值时） |
| 6 | 破坏性重构（Migration） | GPT-5 Codex Max + Claude Opus |

> 详细触发阈值与质量门见 [SOP § 八](./docs/ai-workflow-sop.md#八变更规模阈值可调)。
