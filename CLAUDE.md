---
description: Agent project rules — workflow, task tracking, agent code conventions
alwaysApply: true
---

# Agent Project Rules

> These rules are project-specific and complement the global development rules in `~/CLAUDE.md`.
> Do NOT duplicate global rules here — they are inherited automatically.

---

## 1. Project Awareness & Context

**At the start of every new conversation:**
1. Read `master-plan.md` to understand architecture, phases, tech stack, and constraints
2. Read `CLAUDE.md` (this file) for project-specific conventions
3. Read `PRD.md` for product requirements and feature scope
4. Check `TASK.md` for the current task backlog — if the task you're about to work on isn't listed, add it with a brief description and today's date
5. If phase plans exist in `plans/`, read the relevant phase plan before starting implementation

**Use consistent naming conventions, file structure, and architecture patterns** as described in `master-plan.md` and the global `~/CLAUDE.md`.

---

## 2. Task Tracking (TASK.md)

`TASK.md` is a lightweight, always-current task backlog. It works alongside the formal phase tracking in `plans/[phase]-progress.md`.

**When to update TASK.md:**
- **Before starting work:** Check if your task is listed. If not, add it.
- **After completing a task:** Mark it done immediately with today's date.
- **When discovering new work:** Add sub-tasks or TODOs under a "Discovered During Work" section.
- **When finding bugs:** Log them under a "Bugs" section with a brief description.

**TASK.md structure:**
```markdown
# Task Backlog

## Active
- [ ] [Task description] — [date added]

## Discovered During Work
- [ ] [Task or sub-task discovered mid-implementation] — [date]

## Bugs
- [ ] [Bug description] — [date found]

## Completed
- [x] [Task description] — [date completed]
```

> **Relationship to phase tracking:** `plans/[phase]-progress.md` tracks formal phase execution. `TASK.md` tracks everything else — ad-hoc tasks, bugs, ideas, discoveries.

---

## 3. Agent Code Structure & Modularity

**Organize agent code into clearly separated modules**, grouped by responsibility:

| File | Purpose |
|------|---------|
| `agent.py` | Agent definition, model config, execution entry point |
| `tools.py` | Tool functions the agent can call (each tool = pure function) |
| `prompts.py` | System prompts and prompt templates (string constants) |
| `models.py` | Pydantic models for tool inputs/outputs and agent state |
| `dependencies.py` | Dependency injection (API clients, DB connections, settings) |

**Rules:**
- **Agent code files should stay under 500 lines** — split into sub-modules if approaching this limit
- **Use clear, consistent imports** — prefer relative imports within packages (e.g., `from .tools import search_web`)
- **System prompts NEVER go inline** — always in `prompts.py` as named string constants
- **Each tool function must be independently testable** — pure function with typed inputs/outputs, no hidden state
- **One agent per directory** — if the project has multiple agents, each gets its own directory under `agents/`

---

## 4. Agent Tool Testing

Beyond the global testing rules (3-test minimum, mocking strategy):

- **Always test individual functions for agent tools** — each tool in `tools.py` gets its own test file or test class
- **After updating any agent logic**, check whether existing unit tests need to be updated. If so, update them immediately.
- **Mock all LLM calls and external APIs** in tool tests — never hit real services
- **Test tool input validation** — Pydantic models should reject bad inputs; write tests that verify this

---

## 5. Task Completion Protocol

After completing any implementation task:
1. Mark the task as done in `TASK.md` with today's date
2. If the task was part of a phase, update `plans/[phase]-progress.md`
3. Update `README.md` if: new features added, dependencies changed, setup steps modified, or new env vars introduced
4. Add any newly discovered sub-tasks or follow-ups to `TASK.md` under "Discovered During Work"

---

## 6. Documentation Standards

- **Update `README.md`** whenever the project's setup, usage, or dependencies change
- **Comment non-obvious code** — ensure everything is understandable to a mid-level developer
- When writing complex logic, **add an inline `# Reason:` comment** explaining the *why*, not just the *what*
- Every agent's `prompts.py` should have a module-level docstring explaining the agent's purpose and behavior

---

## 7. Planning Pipeline Reference

This project uses a structured planning pipeline. Never skip steps:

| Step | Command | Artifact |
|------|---------|----------|
| 1 | `/planning/cmo [idea]` | `documents/cmo-thesis.md` |
| 2 | `/planning/cto` | `master-plan.md` + `reference/` guides |
| 3 | `/planning/create-prd` | `PRD.md` |
| 4 | `/planning/create-global-rules` | `CLAUDE.md` (project-specific, overwrites this file) |
| 5 | `/planning/create-reference-guide [url]` | `reference/[category]/[topic].md` |
| 6 | `/planning/plan-phase [phase-N]` | `plans/[phase-name].md` |

**No code is written until ALL planning steps are complete.**

Execution: `/execution/run-phase plans/[phase].md` — orchestrates sub-agents per task.
