# Project Playbook

[English](README.md)


一个 OpenClaw skill，为 AI agent 提供复杂/长链任务的结构化项目管理框架。

## 解决什么问题

AI agent 在执行复杂项目时，经常遇到两个核心问题：
- **上下文溢出** — 随着执行步骤增多，早期的需求和设计决策被挤出上下文窗口
- **注意力漂移** — agent 在执行过程中逐渐偏离初始目标，产出与需求脱节

Project Playbook 通过**持久化文档作为注意力锚点** + **阶段门控** + **机械化 Drift Check**，确保复杂任务从需求到交付的全链路可控。

## 三种模式

| 模式 | 适用场景 | 流程 |
|------|---------|------|
| **L-mode** (轻量) | 3-5 步、单一目标 | 理解 → 计划 → 执行 → 验证 |
| **F-mode** (完整) | >5 步、多模块、需要架构设计 | 5 阶段：需求理解 → 方案设计 → 实施计划 → 执行 → 评审 |
| **C-mode** (长程) | 周期性/循环任务 | 首次走 F-mode，后续周期：执行 → 自检 → 更新状态 |

## 核心机制：Drift Check

每完成一个 task，强制执行 4 项机械化检查：

1. **新增能力检查** — 是否引入了 architecture.md 中未定义的核心能力？
2. **主链纯净检查** — 是否把 extension/fallback 逻辑偷渡进了主链？
3. **契约一致检查** — 输入/输出契约是否与 architecture.md 定义一致？
4. **依赖检查** — 是否新增了外部依赖或人工前置条件？

4 项全 PASS 才能继续。任一 FAIL 触发对应的修复流程。

## 文档锚点

```
{project}/
├── requirements.md    ← 需求文档（Phase 1 产出）
├── architecture.md    ← 方案设计（Phase 2 产出）
├── plan.md            ← 实施计划 + 进度追踪（Phase 3 产出，Phase 4 持续更新）
├── review.md          ← 评审报告（Phase 5 产出）
└── state.json         ← C-mode 专用：周期状态、去重、断点续传
```

## 安装

将 `SKILL.md` 复制到 OpenClaw skills 目录：

```bash
# 共享目录（所有 agent 可用）
cp SKILL.md ~/.openclaw/skills/project-playbook/SKILL.md

# 或 workspace 级别
cp SKILL.md ~/.openclaw/workspace-{name}/skills/project-playbook/SKILL.md
```

重启 OpenClaw 即可生效。

## 许可证

MIT
