# Project Playbook

[中文文档](README_CN.md)


An OpenClaw skill for managing complex and long-running tasks with structured workflows.

## What it does

When AI agents tackle complex projects, they often suffer from **context overflow** (early decisions get pushed out of the context window) and **attention drift** (gradually deviating from the original goal). Project Playbook solves this by using **persisted documents as attention anchors** combined with **phase gates** and **mechanical drift checks**.

## Three Modes

| Mode | When | Flow |
|------|------|------|
| **L-mode** (Lightweight) | 3-5 steps, single goal | Understand → Plan → Execute → Verify |
| **F-mode** (Full) | >5 steps, multi-module, needs architecture | 5-phase: Understand → Design → Plan → Execute → Review |
| **C-mode** (Continuous) | Periodic/recurring tasks | First run: F-mode. Subsequent cycles: Execute → Check → Update state |

## Core Mechanism: Drift Check

After each task completion, a mechanical 4-point checklist is enforced:

1. **New capability check** — Any capabilities not defined in architecture.md?
2. **Main chain purity** — Any extension/fallback logic smuggled into the main path?
3. **Contract consistency** — Input/output contracts match architecture.md?
4. **Dependency check** — Any new external dependencies or prerequisites?

All 4 must PASS to continue. Failures trigger documented remediation.

## Document Anchors

```
{project}/
├── requirements.md    ← What we're building (Phase 1)
├── architecture.md    ← How we're building it (Phase 2)
├── plan.md            ← Task list with status tracking (Phase 3, updated in Phase 4)
├── review.md          ← Final review report (Phase 5)
└── state.json         ← C-mode: cycle state for deduplication & context rebuild
```

## Installation

Copy the `SKILL.md` file to your OpenClaw skills directory:

```bash
# Shared (all agents)
cp SKILL.md ~/.openclaw/skills/project-playbook/SKILL.md

# Or workspace-specific
cp SKILL.md ~/.openclaw/workspace-{name}/skills/project-playbook/SKILL.md
```

Restart OpenClaw to load the skill.

## License

MIT
