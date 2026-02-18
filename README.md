# Agent Project Template

A structured template for AI-assisted development of **AI agent projects** (Python agents + Next.js frontend). Enforces a sequential planning workflow where all strategy, architecture, and requirements are completed before any code is written.

Works with **Claude Code** and **Cursor IDE** — project rules are loaded automatically by both tools.

## Tech Stack

| Layer | Stack |
|-------|-------|
| **Agent Code** | Python 3.12+, Pydantic AI / LangChain, Pydantic v2, httpx |
| **Frontend** | Next.js (latest), TypeScript strict, Tailwind CSS, shadcn/ui |
| **Backend** | Convex or Supabase (decided during planning) |
| **Deployment** | Vercel (frontend + agent serverless functions), Docker (fallback for long-running agents) |
| **Testing** | Pytest (Python), Vitest/Jest (TypeScript) |
| **Linting** | Ruff (Python), Biome (TypeScript) |

## IDE Support

| Tool | Rules File | Scope |
|------|-----------|-------|
| Claude Code | `~/CLAUDE.md` | Global coding standards (all projects) |
| Claude Code | `CLAUDE.md` (project root) | Agent project workflow, task tracking, agent code conventions |
| Cursor | `.cursor/rules/agent-project.mdc` | Agent project rules (self-contained, same content as project CLAUDE.md) |
| Cursor | `.cursor/rules/openmemory.mdc` | OpenMemory integration |

Both IDEs automatically load their respective rule files. No manual configuration needed.

## Workflow

### Phase 1: Planning — NO CODE

Run these commands sequentially. Each validates its prerequisites exist.

| Step | Command | Input | Output | Purpose |
|------|---------|-------|--------|---------|
| 1 | `/planning/cmo [idea]` | Product idea | `documents/cmo-thesis.md` | Market research, competitive analysis, feature prioritization |
| 2 | `/planning/cto` | CMO thesis + CLAUDE_REF.md | `master-plan.md` + `reference/` guides | Architecture, tech stack, phased delivery plan, reference guide collection |
| 3 | `/planning/create-prd` | CMO thesis + master plan | `PRD.md` | Full product requirements document |
| 4 | `/planning/create-global-rules` | PRD + CMO + CTO + CLAUDE_REF.md | `CLAUDE.md` | Project-specific coding standards (overwrites template default) |
| 5 | `/planning/create-reference-guide [url]` | URLs, topics, articles | `reference/[category]/[topic].md` | Store additional docs, articles, patterns (run multiple times) |
| 6 | `/planning/plan-phase [phase-N]` | Master plan phases | `plans/[phase-name].md` | Detailed implementation plan per phase (run for each phase) |

**CRITICAL: No code is written until ALL planning steps (1-6) are complete.**

### Phase 2: Execution — Code Allowed

`/execution/run-phase` is the primary execution command. It acts as a **lightweight orchestrator** that spawns sub-agents for each task.

| Command | Purpose |
|---------|---------|
| `/execution/run-phase plans/[phase].md` | **Orchestrator** — primes context, then spawns sequential sub-agents per task |
| `/execution/prime` | Step 0: Load project context into agent memory |
| `/execution/execute plans/[phase].md [task-N]` | Execute a single task from a plan |
| `/execution/code-review` | Technical code review on changes |
| `/execution/code-review-fix [review-file]` | Fix issues found in code review |
| `/execution/validate` | Run tests, types, lint, build |
| `/execution/execution-report` | Generate implementation report (per-task or per-phase) |
| `/execution/system-review [plan] [report]` | Process improvement analysis (after all tasks complete) |
| `/commit` | Create atomic git commit with conventional tag |

**How `run-phase` works:**

```
1. Prime (once — load codebase context)

FOR EACH TASK (1 to 4, sequentially):
  2. Spawn sub-agent → Execute task
  3. Sub-agent → Code review (task changes only)
  4. Sub-agent → Fix issues (if any)
  5. Sub-agent → Validate (full project)
  6. Sub-agent → Execution report (per-task)
  7. Sub-agent → Commit (atomic, per-task)

8. Spawn sub-agent → System review (whole phase)
```

Each sub-agent gets a fresh context window. Progress is tracked in `plans/[phase]-progress.md` for resilience — re-running `run-phase` picks up where it left off.

### Phase 3: Bug Fix — When Issues Arise

| Command | Purpose |
|---------|---------|
| `/bug-fix/rca [issue-id-or-description]` | Root cause analysis (GitHub issues or manual bugs) |
| `/bug-fix/implement-fix [bug-identifier]` | Implement fix from RCA document |

## Task Tracking

Two complementary systems work together:

| System | File | Purpose |
|--------|------|---------|
| **Phase tracking** | `plans/[phase]-progress.md` | Formal per-task progress during phase execution (managed by `run-phase`) |
| **Task backlog** | `TASK.md` | Lightweight running backlog for ad-hoc tasks, bugs, and discoveries |

`TASK.md` is checked by the AI assistant at the start of every conversation and updated after every task completion. It captures work that falls outside the formal phase pipeline.

## Agent Code Architecture

Agent code is Python-first, organized by responsibility:

```
agents/
  [agent-name]/
    agent.py          # Agent definition, model config, execution entry point
    tools.py          # Tool functions (pure functions with typed inputs/outputs)
    prompts.py        # System prompts and prompt templates (string constants)
    models.py         # Pydantic models for tool inputs/outputs and agent state
    dependencies.py   # Dependency injection (API clients, DB connections, settings)
    __init__.py
  shared/
    logger.py         # Structured JSON logger (shared across agents)
    settings.py       # Pydantic settings for env var management
    exceptions.py     # Custom exception classes
tests/
  agents/
    [agent-name]/
      test_tools.py   # Unit tests for each tool function
      test_agent.py   # Integration tests for the agent
      conftest.py     # Fixtures, mocks, test helpers
```

**Key rules:**
- Agent code files: 500-line max. General code files: 1000-line max.
- System prompts always in `prompts.py`, never inline
- Each tool function is a pure function — independently testable
- All external calls (LLM, APIs, DB) are mocked in tests
- Minimum 3 tests per tool function: happy path, failure case, edge case

## Token Budget

- Each **task** is designed to fit within **150K tokens** (conservative Opus 4.6 budget for a single sub-agent execution cycle)
- Each **phase** has a maximum of **4 tasks**, logically grouped by the CTO
- The `run-phase` orchestrator stays lightweight — sub-agents handle the heavy work
- Progress tracking ensures resilience across long execution sessions

## Environment Variables

- Never hardcode secrets in source code — always use `.env` files
- Always provide a `.env.example` with placeholder values
- Python agents use `pydantic-settings` for env var management
- Frontend uses `process.env` via Next.js

## Directory Structure

```
project-root/
  .claude/commands/             # Slash commands (auto-discovered by Claude Code)
    planning/                   # Steps 1-6: Strategy and planning
      cmo.md                    # Market research & product strategy
      cto.md                    # Technical strategy & master plan
      create-prd.md             # Product requirements document
      create-global-rules.md    # Generate project CLAUDE.md
      create-reference-guide.md # Store reference material
      plan-phase.md             # Phase implementation plans
    execution/                  # Steps 0-6: Code implementation pipeline
      run-phase.md              # Orchestrator — spawns sub-agents per task
      prime.md                  # Load project context
      execute.md                # Implement a single task from plan
      code-review.md            # Technical quality review
      code-review-fix.md        # Fix review issues
      validate.md               # Tests, types, lint, build
      execution-report.md       # Implementation reflection (per-task or per-phase)
      system-review.md          # Process improvement analysis (per-phase)
    bug-fix/                    # Bug investigation and fixing
      rca.md                    # Root cause analysis
      implement-fix.md          # Implement fix from RCA
    commit.md                   # Git commit helper
  .cursor/rules/                # Cursor IDE rules (auto-loaded)
    agent-project.mdc           # Agent project rules for Cursor
    openmemory.mdc              # OpenMemory integration
  .agents/                      # Agent-generated reports (tracked in git)
    code-reviews/               # Code review outputs (per-task)
    execution-reports/          # Implementation reports (per-task)
    system-reviews/             # Process improvement reports (per-phase)
    rca/                        # Root cause analysis documents
  agents/                       # Python agent source code (created during execution)
    [agent-name]/               # One directory per agent
      agent.py                  # Agent definition and entry point
      tools.py                  # Tool functions
      prompts.py                # System prompts
      models.py                 # Pydantic models
      dependencies.py           # Dependency injection
    shared/                     # Shared utilities across agents
  documents/                    # CMO thesis and strategy outputs
  plans/                        # Phase implementation plans + progress files
  reference/                    # Reference guides, articles, documentation
    api-docs/                   # API and SDK documentation
    patterns/                   # Design and architecture patterns
    guides/                     # How-to guides and tutorials
    articles/                   # Blog posts, articles, opinion pieces
  tests/                        # Test suite (mirrors agent source structure)
    agents/                     # Agent tests
  prompts/                      # Copy-paste prompts for AI assistants
    usable_prompts/             # Layer 1 context building prompts
  src/                          # Frontend source code (created during execution)
    components/                 # React components
    lib/                        # Utilities and helpers
    types/                      # TypeScript type definitions
  CLAUDE_REF.md                 # Reference template for global rules
  CLAUDE.md                     # Project-specific rules (template default; overwritten by /create-global-rules)
  TASK.md                       # Lightweight task backlog (always current)
  master-plan.md                # CTO master plan (generated)
  PRD.md                        # Product requirements (generated)
  .env.example                  # Environment variable template (generated during execution)
```

## Quick Start

1. Clone this template for your new project
2. Ensure `~/CLAUDE.md` has the global development rules (shared across all projects)
3. Run `/planning/cmo [your product idea]` to start market research
4. Follow the planning workflow steps sequentially (1 through 6)
5. Run `/execution/run-phase plans/[phase-name].md` to execute each phase
6. Track ad-hoc tasks and discoveries in `TASK.md`
7. Each task is committed atomically — review progress in `plans/[phase]-progress.md`
