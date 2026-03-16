# AI 辅助开发工作流 SOP（可执行版）

> **适用场景**：文档设计 → 任务拆解 → 长周期代码实施 → 多轮需求变更 → 可能的破坏性重构。  
> **版本**：v1.0 | **最后更新**：2026-03-16

---

## 一、总体原则（必读）

| 原则 | 说明 |
|------|------|
| 低成本优先 | 日常高频交互优先使用低延迟、低成本模型（GPT-4 Turbo / Claude Haiku / Gemini Flash 等）。 |
| 大模型里程碑专用 | 里程碑级决策、跨仓库/破坏性重构使用大上下文、推理强的模型（Gemini Pro / GPT-5 Codex Max / Claude Opus）。 |
| 人工审核不可省 | 所有模型输出必须经人审核；重大变更（见§八阈值）需经过设计复审与 CI Canary 测试。 |
| 版本化决策 | 把每次重要 AI 决策版本化并写入 [DECISIONS.md](../DECISIONS.md) / [DESIGN.md](../DESIGN.md)；作为未来 prompt 的上下文来源。 |

---

## 二、阶段化 SOP

### 阶段 1：需求收集（Discovery / Intake）

**目标**：把会议记录、需求文档、原型、现有代码快照整合成"问题清单、关键假设与风险清单"。

| 项目 | 内容 |
|------|------|
| 推荐主模型 | Gemini Pro（文档量大时）或 Claude Sonnet |
| 备选模型 | GPT-4o（含图片/原型时） |
| 输入 | 所有 docs + 最近 7 天变更 diff + 关键截图（如有） |
| 输出（必有） | **Project Intake Summary**（YAML/MD）：关键功能、约束、未定项、澄清问题清单、初步风险评级（低/中/高） |
| 触发大模型条件 | 文档总 token 估算 > 100k，或包含多种媒介（大量图片/PDF）时用 Gemini；否则 Claude Sonnet 足矣 |

**Intake Summary 输出格式（YAML）**：

```yaml
intake_summary:
  project: "<项目名>"
  date: "<YYYY-MM-DD>"
  key_features:
    - "<功能1>"
    - "<功能2>"
  constraints:
    - "<约束1>"
  open_questions:
    - question: "<问题>"
      source_doc: "<文档名/段落>"
  risks:
    - description: "<风险描述>"
      level: "低 | 中 | 高"
      mitigation: "<缓解措施>"
```

---

### 阶段 2：高层架构与总体规划

**目标**：生成 2–3 个候选架构、选型理由、非功能需求影响、回滚策略。

| 项目 | 内容 |
|------|------|
| 推荐主模型 | Claude Opus 或 GPT-5 Codex Max |
| 备选模型 | Gemini Pro（跨文件依赖分析） |
| 输入 | Intake Summary + 技术栈约束 + SLA/预算/团队规模 |
| 输出（必有） | **Architecture Decision Record (ADR)**：方案 A/B/C、优劣、实施成本估计（人周）、迁移/回滚路径、关键风险 |
| 审批 | 至少 2 人（架构师 + PM）签署后方可进入拆解阶段 |
| 调用策略 | 一次性里程碑调用；保存模型生成内容作为 ADR v1（见 [adr-template.md](./adr-template.md)） |

---

### 阶段 3：任务拆解与里程碑（Planning）

**目标**：将 ADR 拆为 Epics → Stories → Tasks，并生成验收标准与估时。

| 项目 | 内容 |
|------|------|
| 推荐主模型 | GPT-4o 或 o3-mini |
| 备选模型 | Claude Sonnet（更细化的验收标准） |
| 输入 | ADR + 代码库最小上下文（相关模块文件摘要） |
| 输出 | Roadmap/Backlog（GitHub Issues 批量 JSON 或 Jira Ticket 列表）：目标、输入、输出、验收标准、依赖、估时、风险等级 |
| 自动化 | 生成 Issues 提交脚本（带模板：PR 标题、描述、测试要点） |

**Backlog Item 示例格式（JSON）**：

```json
{
  "epic": "<Epic 名称>",
  "story": "<Story 描述>",
  "tasks": [
    {
      "title": "<任务标题>",
      "goal": "<目标>",
      "inputs": ["<输入文件/接口>"],
      "outputs": ["<输出文件/接口>"],
      "acceptance_criteria": ["<验收标准1>", "<验收标准2>"],
      "dependencies": ["<依赖任务 ID>"],
      "estimate_hours": 4,
      "risk_level": "低 | 中 | 高"
    }
  ]
}
```

---

### 阶段 4：日常开发（Sprint / Week-to-week）

**目标**：日常补全、实现、测试、Code Review 支持。

| 项目 | 内容 |
|------|------|
| 推荐主模型 | GPT-4 Turbo / Gemini Flash / Claude Haiku / Grok Code Fast（JS/TS） |
| 使用方式 | IDE/Chat 短会话、高频调用；开发者在 prompt 中提供当前文件上下文（≤ 2–4 KB） |
| 输出 | 补全片段、单元测试、修复建议、可执行代码片段 |
| 质量控制 | 每个 AI 生成的实现都要带自动生成的单元测试并通过本地 CI 才能合并 |

**日常开发 Prompt 最佳实践**：

```
# 上下文（≤ 2–4 KB）
文件：<path/to/file>
当前函数：<函数签名>

# 任务
请帮我实现 <功能描述>。
要求：
- 不引入新依赖
- 保持现有接口不变
- 生成对应的单元测试（放入 <test/path>）
```

---

### 阶段 5：需求变更与增量调整（Iterative Replanning）

**目标**：快速评估变更影响并给出调整计划或重构建议。

| 项目 | 内容 |
|------|------|
| 快速评估模型 | GPT-4o / Turbo（初步影响分析） |
| 深度分析模型 | Gemini Pro / GPT-5 Codex Max / Claude Opus（影响超出阈值时） |
| 输出 | **Impact Analysis**：受影响文件/接口清单、测试盲点、迁移方案、估时 |

**触发大模型的判定阈值**：

- [ ] 估计受影响文件 **> 30**；或
- [ ] 涉及数据库 schema / API contract 更改；或
- [ ] 改动跨 **≥ 3 子系统/模块**；或
- [ ] 变更引入向后不兼容（breaking change）

**执行策略**：若为破坏性变更，先执行小范围 PoC（单模块），在 CI 上做 Canary 测试并监测关键指标。

---

### 阶段 6：破坏性重构 / 全仓库迁移

**目标**：安全、可回滚地完成重构或迁移。

| 项目 | 内容 |
|------|------|
| 推荐主模型 | GPT-5 Codex Max + Claude Opus（并行：前者生成迁移脚本，后者负责策略/风险评估） |
| 备选 | Gemini Pro（全仓库依赖分析） |

**分阶段执行**：

1. **快照与指标定义**：保存 baseline、制定回测指标与回滚条件。
2. **生成迁移蓝图**：模型产出分步计划。
3. **试点**：小范围自动 PR 与 CI 测试。
4. **自动化迁移脚本**：生成带回滚脚本的迁移工具。
5. **批量执行**：分批合并 + Canary + 监控。

**输出**：迁移脚本（含测试）、批量 PR、回滚脚本、监控/回归报告。

> ⚠️ **安全门**：任何无法自动回滚的迁移需经高级审批（≥ 2 人，含 SRE/安全负责人）。

---

## 三、CI / Automation 示例

CI 工作流配置见 [`.github/workflows/ai-generated-pr-check.yml`](../.github/workflows/ai-generated-pr-check.yml)。

**流程说明**：
- `ai-wrapper` 为内部工具：封装模型选择、上下文裁剪、prompt 注入与审计日志。
- 可用企业内部 API 或脚本替代（将模型类型作为参数传入）。
- 默认日常 PR 使用低延迟模型（Turbo/Haiku/Flash），里程碑 PR 升级为大模型。

---

## 四、上下文管理与成本控制

### 上下文管理

- 使用向量数据库（embeddings）建立"模块级片段索引"；调用时只把最相关 10–20 个片段作为上下文（节省 token）。
- 每次重大变更，把摘要写入 `DECISIONS.md` 与 `DESIGN.md`，在后续 prompts 里引用 `DECISION:vX`。
- 为长会话维护"会话快照"：`session_id` + `latest_decision_refs`（避免重复发送历史）。

### 成本控制规则

| 场景 | 策略 |
|------|------|
| 日常 IDE/Chat | 默认低成本模型（Turbo/Haiku/Flash）；每人每天配额限制 |
| 里程碑/重构调用 | 需在 PR/Issue 中申请审批，由项目预算扣费 |
| 重复请求 | 相同 prompt + 相同上下文，先查缓存/摘要结果再发请求 |
| 监控 | 每周报告模型调用次数/token 用量/花费与成功率 |

---

## 五、审计与合规

- 所有调用记录（input prompt 摘要、模型版本、response id、cost）必须写入审计日志（自动化）。
- 含敏感信息（密钥、个人数据）的上下文，必须先脱敏或使用企业合规模式（如 Claude Enterprise）。

---

## 六、审核、回滚与质量门（必须执行）

### 合并前门控

1. 所有 AI 生成代码必须包含自动生成的单元测试并通过本地/CI 测试。
2. 代码变更 **> 10 文件**必须经过人工 Code Review + Architecture Owner 批准。
3. 破坏性变更需提供回滚脚本与 Canary 计划。

### 回滚策略

| 类型 | 触发条件 | 执行方式 |
|------|----------|----------|
| 自动回滚 | CI 未通过、关键监控指标异常、SLA 降级 | 自动触发 revert pipeline |
| 人工回滚 | 需人工判断的异常 | PR 描述中包含 `revert commit` 指令与回滚负责人 |

---

## 七、团队角色与职责

| 角色 | 职责 |
|------|------|
| **Product Owner** | 确认需求、触发重大模型调用审批、签署 ADR |
| **Tech Lead / Architect** | 评审架构、批准破坏性迁移 |
| **Dev** | 日常使用低延迟模型、写测试、执行 CI |
| **SRE/QA** | 设计 Canary 流程、监控指标、执行回滚 |
| **AI Steward (DevOps/Platform)** | 维护 ai-wrapper、审计日志、成本仪表盘、模型访问控制 |

---

## 八、变更规模阈值（可调）

| 规模 | 条件 | 推荐模型 | 流程 |
|------|------|----------|------|
| **小变更** | 受影响文件 ≤ 10、单一模块 | GPT-4 Turbo / Claude Haiku | 常规 PR + CI |
| **中等变更** | 11–30 文件、跨 2–3 模块、涉及接口扩展 | GPT-4o / o3-mini / Claude Sonnet | PR + 人工 Review |
| **大变更** | > 30 文件、跨 ≥ 3 模块、DB schema 或 breaking API | Gemini Pro / GPT-5 Codex Max / Claude Opus | PoC → Canary → 批量合并 |

---

## 九、运维工具建议

| 工具 | 用途 |
|------|------|
| `ai-wrapper`（内部） | 统一入口：模型选择策略、prompt 模板、上下文裁剪、结果缓存、审计上报 |
| Embeddings + Vector DB（Milvus/Weaviate/Chroma） | 模块级片段检索，减少 token 传输 |
| Decision Store（版本化 DECISIONS.md API） | 每次重要模型输出创建条目，供后续 prompt 引用 |
| Billing & Quota Dashboard | 按项目/团队统计模型调用与成本 |

---

## 十、重大调用前 Checklist（每次必须执行）

- [ ] Intake 文档已汇总并写入 Intake Summary。
- [ ] Prompt 模板已选定并验证（包含上下文裁剪）。
- [ ] 预算/配额审批（若调用大模型）。
- [ ] 输出（ADR/Impact Analysis）版本化并存档。
- [ ] 测试/回滚/Canary 流程已定义。
- [ ] 审批人签名（PR 评论或 Jira 批注）。

---

## 相关文档

- [Prompt 模板集](./prompt-templates.md)
- [ADR 模板](./adr-template.md)
- [DECISIONS.md](../DECISIONS.md)
- [DESIGN.md](../DESIGN.md)
- [CI 工作流](./../.github/workflows/ai-generated-pr-check.yml)
