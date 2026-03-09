---
name: project-playbook
description: >
  复杂/长链任务的结构化项目管理框架。当任务涉及多步骤实施、架构设计、
  长程持续执行时触发。提供 需求理解→方案设计→实施计划→执行→评审 的
  完整闭环，用持久化文档对抗上下文溢出和注意力漂移。

  触发场景：
  (1) 多文件多步骤的项目级任务（新功能开发、重构、迁移）
  (2) 需要架构设计的系统级任务
  (3) 周期性长程持续任务（信息收集、数据跟踪、定期报告）
  (4) 用户明确要求"按流程来"、"做个计划"、"先别急着写代码"

  不触发：单文件修改、简单查询、已有专用 skill 覆盖的场景（如 gh-issues）
---

# Project Playbook

复杂/长链任务的结构化管理框架。用持久化文档作为注意力锚点，通过阶段门控和 drift check 对抗上下文溢出和注意力漂移。

## Step 0: 复杂度评估与路径选择

收到任务后，先评估复杂度，选择路径：

**评估维度：**
- 预估步骤数
- 涉及文件/模块数
- 是否需要架构决策
- 时间跨度（单次 vs 跨 session/周期）
- 依赖关系复杂度

**路径判定：**

| 条件 | 路径 |
|------|------|
| 3-5 步、单一目标、无需架构设计 | **L-mode** (轻量) |
| > 5 步、多文件/多模块、需要架构决策 | **F-mode** (完整) |
| 周期性/循环任务、跨 session 持续执行 | **C-mode** (长程) |

如果不确定，默认选 F-mode。宁可多做计划，不要少做。

选定路径后，在工作目录下创建项目目录：
```bash
mkdir -p {project-name}
```

---

## L-mode (轻量路径)

适用于中等复杂度任务。跳过架构设计，直接做计划和执行。

### L1: 理解
用 1-2 段话总结任务目标和边界，写入 `plan.md` 顶部。

### L2: 计划
在 `plan.md` 中列出 checklist：

```markdown
# Plan: {项目名}
## 目标
{一段话}
## Checklist
- [ ] Step 1: ...
- [ ] Step 2: ...
- [ ] Step 3: ...
## 完成标准
{怎么算做完}
```

### L3: 执行
逐项完成，完成一项勾一项。

### L4: 验证
全部完成后，回读 plan.md 顶部的目标，确认产出匹配。

---

## F-mode (完整路径)

适用于复杂项目级任务。完整 5 阶段流程。

### Phase 1: 需求理解 (Understand)

**做什么：** 拆解需求、识别场景、明确边界、确认约束。

**产出：** `requirements.md`

```markdown
# Requirements: {项目名}

## 核心问题
{一句话描述要解决什么}

## 目标
{要达成的具体结果}

## 场景
{具体使用场景，谁在什么情况下用}

## 不做什么
{明确排除的范围}

## 约束
{技术/资源/时间约束}

## 验收标准
{可检验的完成条件}
```

**门控：** 用户确认 requirements.md 内容无误后，才进入 Phase 2。

> 门控可跳过的条件：用户明确要求"端到端推进"/"全自动"，或 C-mode 后续周期执行。此时 agent 自行验证 requirements.md 完整性后直接进入下一阶段。

---

### Phase 2: 方案设计 (Design)

**输入：** 回读 `requirements.md`

**做什么：** 设计架构、选择技术方案、定义模块边界、识别风险。

**产出：** `architecture.md`

```markdown
# Architecture: {项目名}

## 设计总览
{整体方案一段话}

## 模块划分
| 模块 | 职责 | 输入 | 输出 |

## 关键设计决策
| 决策 | 选项 | 选择 | 理由 | 对应需求 |

## 风险与缓解
| 风险 | 影响 | 缓解策略 |
```

**门控：** architecture.md 中每个设计决策都能追溯到 requirements.md 中的需求项。无法追溯的决策 = 超出范围，删除或补充需求。

---

### Phase 3: 实施计划 (Plan)

**输入：** 回读 `architecture.md`

**做什么：** 将架构拆解为可执行的 task list，定义依赖关系和优先级。

**产出：** `plan.md`

```markdown
# Plan: {项目名}

## Task List
| # | Task | 对应模块 | 依赖 | 状态 | 备注 |
|---|------|---------|------|------|------|
| T1 | ... | arch.§X | - | pending | |
| T2 | ... | arch.§Y | T1 | pending | |

## 反向追溯
| Architecture 章节 | 覆盖 Task |
|-------------------|----------|
| §X | T1, T3 |
| §Y | T2 |

状态值: pending → in-progress → done → verified
特殊: blocked({原因}) / skipped({原因})
```

**门控：**
- 正向：每个 task 映射到 architecture.md 的具体章节
- 反向：architecture.md 每个关键章节都被至少一个 task 覆盖

---

### Phase 4: 执行 (Execute)

**输入：** 回读 `plan.md` + `architecture.md`

**做什么：** 按 plan 逐项执行。每完成一个 task（或每 3 个，取决于复杂度），执行 Drift Check。

**Drift Check（核心机制）：**

回读 architecture.md 相关章节，逐项检查：

```
Drift Check Checklist (4 项全 PASS 才继续):

□ 1. 新增能力检查 — 是否引入了 architecture.md 中未出现的核心能力？
□ 2. 主链纯净检查 — 是否把 extension/fallback 逻辑偷渡进了主链？
□ 3. 契约一致检查 — 当前阶段的输入/输出契约是否与 architecture.md 定义一致？
□ 4. 依赖检查 — 是否新增了外部依赖或人工前置条件？
```

**判定：**
- 4/4 PASS → 更新 plan.md 状态为 verified，继续下一个 task
- FAIL 且可接受 → 记录到 plan.md 备注列，标注理由，继续
- FAIL 且需修正 → 回退修复，更新 plan.md 状态
- FAIL 且需改架构 → 暂停执行，显式记录变更理由到 architecture.md，重新评估后续 plan

**产出：** 实际产出物 + 持续更新的 plan.md

---

### Phase 5: 评审 (Review)

**输入：** 回读 `requirements.md` + `architecture.md` + 所有产出物

**做什么：** 逐项对照需求和架构，验证完整性和一致性。

**产出：** `review.md`

```markdown
# Review: {项目名}

## 需求覆盖检查
| 需求项 | 状态 | 对应产出 | 备注 |

## 架构一致性检查
| 设计决策 | 实际实现 | 一致? | 偏差说明 |

## 偏差汇总
{执行过程中的所有偏差及处理}

## 遗留问题
{未解决的问题}

## 结论
{通过 / 有条件通过 / 需返工}
```

**门控：** 所有需求项都有对应产出，所有偏差都有记录和理由。

---

## C-mode (长程持续路径)

适用于周期性/循环任务。首次执行走 F-mode 全流程，后续周期走简化循环。

### 首次执行

按 F-mode 完整走一遍 Phase 1-5，额外创建 `state.json`：

```json
{
  "project": "{项目名}",
  "mode": "continuous",
  "cycle": {
    "current": 1,
    "last_run": "{ISO 时间}",
    "next_scheduled": "{ISO 时间}"
  },
  "processed": {
    "description": "已处理数据的标识，用于去重",
    "items": []
  },
  "checkpoint": {
    "phase": "complete",
    "last_task": "T{n}",
    "notes": ""
  }
}
```

### 后续周期

每个新周期按以下顺序重建上下文（顺序刻意设计，先定位再加载细节）：

1. **读 state.json** — 知道"我在哪"（当前周期、上次进度、已处理数据）
2. **读 requirements.md** — 知道"我要去哪"（确认目标未变）
3. **读 architecture.md 相关章节** — 知道"怎么去"
4. **读 plan.md 当前进度** — 知道"下一步做什么"

然后进入执行循环：
```
执行 plan 中的待办 task
    → drift check
    → 更新 plan.md 状态
    → 更新 state.json（processed items + checkpoint）
    → 周期结束，等待下次触发
```

**去重：** 每次处理数据前，检查 state.json.processed.items，跳过已处理项。处理完新数据后，将其标识追加到 items 中。

---

## 与 coding-agent 的协作

project-playbook 负责"想清楚"，coding-agent 负责"做出来"：

- **Phase 1-3** 由 playbook 驱动，不需要 coding-agent
- **Phase 4** 执行时，如果是 coding 任务，将当前 task 的描述 + architecture.md 相关章节作为 prompt 传给 coding-agent
- **Phase 5** 评审由 playbook 驱动，可调用 coding-agent 跑测试验证

传递格式：
```
Task: {task 描述}
Context: {architecture.md 相关章节}
Constraints: {从 requirements.md 提取的约束}
Working directory: {项目目录}
```
