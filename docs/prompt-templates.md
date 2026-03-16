# Prompt 模板集（可直接复制使用）

> 这些模板与 [AI 辅助开发工作流 SOP](./ai-workflow-sop.md) 配合使用。  
> 复制时将 `<...>` 占位符替换为实际内容。

---

## 模板 1：Intake Summarizer（需求收集汇总）

**使用阶段**：§ 阶段 1 — 需求收集  
**推荐模型**：Gemini Pro（文档量大）或 Claude Sonnet

```
这是项目 Intake 列表：

<list of docs + short excerpts>
例：
- requirements.md（节选）：...
- meeting-notes-2026-03-01.md（节选）：...
- prototype-screenshot.png（描述）：...

请输出：
1) 关键功能点（bullet 列表）
2) 未明确/冲突需求（bullet 列表，标注来源文档）
3) 风险清单（每项带风险等级：低/中/高）
4) 需要澄清的问题列表（每项一行，指向具体文档/段落）

格式：YAML，字段参考：
intake_summary:
  key_features: []
  unclear_requirements: []
  risks: []
  clarification_questions: []
```

---

## 模板 2：高层架构决策（Architecture ADR）

**使用阶段**：§ 阶段 2 — 高层架构与总体规划  
**推荐模型**：Claude Opus 或 GPT-5 Codex Max

```
约束条件：
- 技术栈：<栈信息，如 Node.js 18 + PostgreSQL + React>
- SLA：<如 99.9% uptime，P99 < 200ms>
- 团队规模：<如 3 名后端 + 2 名前端>
- Timeline：<如 12 周>

请基于以下 Intake Summary 生成 3 个架构方案：

<粘贴 Intake Summary YAML>

每个方案请提供：
- 方案描述（2–3 句）
- 优点（bullet）
- 缺点（bullet）
- 实施成本估计（人周）
- 迁移风险（低/中/高 + 说明）
- 回滚策略

最后给出推荐方案并说明理由。

输出格式：ADR Markdown（参见 adr-template.md）。
```

---

## 模板 3：任务拆解（Planning Breakdown）

**使用阶段**：§ 阶段 3 — 任务拆解与里程碑  
**推荐模型**：GPT-4o 或 o3-mini

```
请将以下 ADR 拆解为可执行任务：

<粘贴 ADR 推荐方案部分>

代码库相关模块摘要：
<粘贴相关模块文件摘要，≤ 2000 token>

拆解格式：Epics → Stories → Tasks
每个 Task 包含：
- title: 任务标题
- goal: 目标（一句话）
- inputs: 输入文件/接口列表
- outputs: 输出文件/接口列表
- acceptance_criteria: 验收标准列表
- dependencies: 依赖任务 ID 列表
- estimate_hours: 估时（小时）
- risk_level: 低 | 中 | 高

输出格式：JSON 列表（可用于批量创建 GitHub Issues）。
```

---

## 模板 4：影响分析（需求变更）

**使用阶段**：§ 阶段 5 — 需求变更与增量调整  
**推荐模型**：GPT-4o（初步）→ 大模型（超出阈值时）

```
变更说明：
<描述变更内容，例：将用户认证从 JWT 迁移到 OAuth2.0>

相关代码上下文：
<粘贴相关文件/接口摘要，≤ 3000 token>

请分析：
1. 受影响模块/文件清单（粗估）
2. 潜在破坏点（API contract / DB schema / 第三方集成）
3. 需要新增或修改的测试场景
4. 分步迁移建议（含回滚点）
5. 估时（小时）

若受影响文件 > 30，请在输出顶部标注：
⚠️ 建议升级为全仓库大模型分析

输出格式：JSON
{
  "affected_files": [],
  "breaking_points": [],
  "tests_needed": [],
  "migration_steps": [],
  "rollback_points": [],
  "estimate_hours": 0,
  "upgrade_recommended": false
}
```

---

## 模板 5：迁移脚本生成（破坏性重构）

**使用阶段**：§ 阶段 6 — 破坏性重构 / 全仓库迁移  
**推荐模型**：GPT-5 Codex Max 或 Gemini Pro

```
目标：将 <old API / 旧实现> 替换为 <new API / 新实现>。

相关代码片段：
<粘贴关键代码片段，≤ 4000 token>

请生成：
1. 一个可运行的批量替换脚本（bash 或 Python）
   - 必须支持 --dry-run 模式（只输出变更列表，不实际修改）
   - 输出每个修改文件的 diff 摘要

2. 每个自动生成 PR 的描述模板（Markdown），包含：
   - 变更摘要
   - 影响范围
   - 测试说明
   - 回滚方法

3. 回滚脚本（可一键恢复到迁移前状态）

4. 最小化 CI 检查清单（迁移验证需通过哪些测试/指标）
```

---

## 模板 6：代码审查辅助（Code Review）

**使用阶段**：§ 阶段 4 — 日常开发（PR Review 支持）  
**推荐模型**：GPT-4 Turbo / Claude Haiku（快速）

```
请对以下代码变更进行审查：

<粘贴 PR diff，≤ 3000 token>

审查重点：
1. 正确性（逻辑错误、边界情况）
2. 安全性（SQL 注入、XSS、敏感数据暴露等）
3. 性能（明显的 N+1、不必要的循环、内存泄漏）
4. 可维护性（命名、函数长度、注释）
5. 测试覆盖（是否覆盖主要路径和边界情况）

每个问题请标注：严重程度（阻塞 / 建议 / 可选）+ 具体位置 + 改进建议。
```

---

## 使用说明

1. 复制对应模板，替换 `<...>` 占位符。
2. 控制上下文长度（参考各模板注释中的 token 建议）。
3. 将模型输出版本化：保存到 `DECISIONS.md`（重要决策）或对应 ADR 文件。
4. 重复使用相同 prompt + 上下文时，先检查是否有缓存结果可复用。
