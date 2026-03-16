# GitHub + Copilot 协作 SOP（完整版）

## 目的
- 明确"讨论 ↔ 执行"分工，避免误触发 Copilot 执行造成额外 premium requests / Actions minutes 消耗。
- 提高 PR 质量，减少返工次数与 CI/CD 成本。

## 适用范围
- Private 仓库（GitHub Free + Copilot Pro+）开发流程
- 团队中同时使用人工 review 与 Copilot coding agent 的情景

## 角色与职责
- 发起人（Author）：提出需求、创建初版 PR、与团队/Assistant 讨论设计。
- 团队 Reviewer：参与设计讨论、给出 review 意见、在讨论达成一致后触发执行。
- GitHub Copilot（agent）：仅在收到明确的 `@copilot` 执行指令时进行代码修改/提交。
- Chat Assistant（即"AI 助手"）：承担讨论、方案比较、把模糊需求翻译为可执行指令、生成 PR 评论模板与 `@copilot` 指令。

## 总体原则（必须遵守）
1. 讨论期间：不得 `@copilot`、不得在评论中包含"请修改这个 PR"之类会触发执行的短语。所有模糊需求在 Chat Assistant（或团队讨论）中解决完毕再进入执行阶段。
2. 执行请求：必须由有写权限的人在 PR Conversation 发出并以 `@copilot` 开头的明确任务单（见模板）。
3. 一次性把要点说清，优先减少返工；尽量用 included models 在 Chat 中讨论以节省 premium requests。
4. CI/CD 尽量轻量化（避免每次提交跑全量矩阵），合并/发布时再跑完整流程。

## 标准流程（Step-by-step）
1. 需求 & 初版实现
   - Author 在本地实现或让 Copilot 创建初版 PR（可以是 draft）。
   - PR 描述写清当前目标、边界情况与待讨论点。

2. 讨论阶段（"不执行"）
   - 所有设计讨论先在 Chat Assistant 或 PR conversation 里进行，但必须明确标注"先不要修改代码"。
   - 启动讨论时在 PR 评论中使用 DISCUSSION 模板或在团队 Chat 中以标准格式启动（见模板）。
   - Chat Assistant 帮助产出：问题清单、对比方案、推荐方案、风险点、验收标准。

3. 确认方案（形成可执行结论）
   - 团队达成一致后，生成两份输出：
     1) 讨论总结 — 记录结论（不触发 Copilot）
     2) `@copilot` 执行指令 — 明确任务单（触发 Copilot）

4. 执行阶段（触发 Copilot）
   - 由有写权限的人在 PR Conversation 发出 `@copilot` 指令（使用模板，列出修改项、约束、验收标准）。
   - 观察 Copilot 是否响应（PR timeline 会显示 agent 开始工作 / 评论上出现 👀 等）。
   - 若需要多轮修改，尽量把 steering comments 合并为一条或在同一 session 内完成，减少多次计费。

5. CI / Review / Merge
   - Copilot 提交后触发 CI（尽量限制为必要 job）。
   - Reviewer 验收测试通过后合并。
   - 合并触发 CD（仅在需要时触发重型部署）。

6. 事后记录与成本复盘（每周或每月）
   - 统计 premium requests、Actions minutes、触发 Copilot 的 PR 数。
   - 若高频返工，改进讨论模板与审查流程。

## 关键模板（可直接复制使用）

### A. 启动讨论（Chat / PR，明确不执行）

```
[DISCUSSION ONLY - DO NOT IMPLEMENT YET]

仓库：<owner/repo>
PR：#<编号> / <PR URL>
关注文件：
- <path/to/file1>
- <path/to/file2>

目标：先讨论，不执行任何代码修改。

当前待确认点：
1. <待确认项1>
2. <待确认项2>
3. <待确认项3>

期望输出：
- 对以上问题的分析
- 推荐方案（含优缺点）
- 风险点与验收标准

等我们确认后，再发一个 @copilot 指令进入执行阶段。
```

### B. 讨论总结（发到 PR，记录，不触发 Copilot）

```
讨论结论（摘要）：

1. 最终决定：
   - <明确结论1>
   - <明确结论2>

2. 需要在执行阶段完成的具体项：
   - a. <修改项 1>
   - b. <修改项 2>

3. 验收标准：
   - 测试覆盖：<列出测试场景>
   - 性能 / 复杂度限制：<如 O(n) 等>

备注：本条为讨论记录，不要求立即修改。执行由 Author/Reviewer 在确认后发起 @copilot。
```

### C. 触发 Copilot 的执行指令（必须由有写权限的人发出）

```
@copilot 请根据我们已确认的讨论结果更新这个 PR。

目标：
- <目标1：例 保持 API 不变 / 提升可读性 / 补测试 等>

请执行以下修改：
1. <修改项一，越具体越好>
2. <修改项二>
3. <修改项三>

约束：
- 不要修改对外导出名称 <如 TreeNode / sumOfAdjacentTriplets>
- 不引入新依赖
- 修改后现有测试必须通过；新增测试请放在 <path/to/testfile>

验收标准：
- 所有 tests 通过
- 在 PR 描述中添加"改动摘要 + 时间复杂度说明"

完成后请：
1. 更新此 PR
2. 在 PR comment 中说明"改了什么 & 为什么改"
```

### D. 只补测试

```
@copilot 请仅补充测试，不要重构现有实现。

需要新增测试场景：
1. 偏斜树（只有左子链）
2. 包含负数与零的节点值
3. 树为空 / 单节点 / 两节点情况

要求：
- 测试命名清晰
- 不改现有实现逻辑
```

## 监控与成本控制规则（必须遵守）
- 讨论优先在 Chat Assistant 或 internal chat 完成，尽量使用 included models（节省 premium requests）。
- 触发 Copilot 时务必使用"执行指令"模板，任务越具体越少返工。
- CI 优化：对 PR 设置快速 smoke-check（短跑），合并前再跑完整矩阵。
- 设置成本阈值告警（Owner）：
  - premium requests 使用阈值：80%、90% 通知
  - Actions minutes 使用阈值：80%、90% 通知
- 记录每次 Copilot 执行次数与对应 PR，供月度复盘。

## 常见问题（FAQ）
- Q：不小心 @copilot 了怎么办？  
  A：立即在 PR 留言"仅讨论，不要执行"，并检查 agent 是否已开始操作；若已执行，人工回滚或修复并记录消耗。

- Q：多轮 steering comment 会重复扣费吗？  
  A：active session 内与模型交互会继续消耗 premium requests；尽量把 steering 意见合并为一条。

- Q：CI minutes 快用完怎么办？  
  A：优先优化 workflow（只在合并时跑完整矩阵），或升级 plan / 使用 self-hosted runners。

## 上手 & on-boarding checklist
1. 团队成员保存并熟悉本 SOP 与模板。
2. 新成员演练一次完整流程。
3. 在仓库 README/CONTRIBUTING 中加入 SOP 摘要与链接。
4. 设置 Cost Alert（GitHub billing）或第三方监控，保证 Actions minutes 与 premium requests 可见。

---
