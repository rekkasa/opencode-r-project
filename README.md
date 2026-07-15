# R Development with Opencode

Personal opencode setup for structured R development, using
custom agents and skills to drive a pipeline from planning
through implementation to tested code.

## Quick Start

```bash
git clone <this-repo>
cd opencode-r-project
ln -s ~/my-skills/.config/opencode/skills/r-style ~/.config/opencode/skills/r-style
ln -s ~/my-skills/.config/opencode/skills/code-implementation-plan ~/.config/opencode/skills/code-implementation-plan
ln -s ~/my-skills/.config/opencode/skills/generate-objectives ~/.config/opencode/skills/objectives
ln -s ~/my-skills/.config/opencode/skills/grill-me ~/.config/opencode/skills/grill-me
```

## How This Works

Opencode loads `opencode.jsonc` at the project root, which sets
`project-lead` as the default agent. That agent orchestrates a
pipeline of subagents — each with a strict domain boundary —
guided by skills loaded from `~/.config/opencode/skills/`.

The pipeline turns a natural-language description into a
grilled-down specification, an implementation plan, working R
code, and a test suite, all following the conventions in
`.project/STYLE_GUIDE.md`.

## Prerequisites

- [Opencode CLI](https://opencode.ai) (latest)
- R >= 4.3
- R packages: tidyverse, devtools, testthat, roxygen2

## Installation

1. **Clone the repository**

   ```bash
   git clone <this-repo> opencode-r-project
   cd opencode-r-project
   ```

2. **Symlink skills**

   The four skills live in `~/my-skills/.config/opencode/skills/`.
   Symlink them into the active opencode skills directory:

   ```bash
   mkdir -p ~/.config/opencode/skills
   ln -s ~/my-skills/.config/opencode/skills/r-style \
     ~/.config/opencode/skills/r-style
   ln -s ~/my-skills/.config/opencode/skills/code-implementation-plan \
     ~/.config/opencode/skills/code-implementation-plan
   ln -s ~/my-skills/.config/opencode/skills/generate-objectives \
     ~/.config/opencode/skills/objectives
   ln -s ~/my-skills/.config/opencode/skills/grill-me \
     ~/.config/opencode/skills/grill-me
   ```

## Project Structure

```
.
├── opencode.jsonc                  Entry point (sets default_agent)
├── .project/
│   ├── STYLE_GUIDE.md              R coding conventions (canonical)
│   ├── .gitignore                  Ignores ARCHITECTURE.md
│   ├── OBJECTIVES.md               [runtime] Feature spec
│   ├── ARCHITECTURE.md             [runtime] Module-level design decisions
│   ├── TASKS/                      [runtime] Per-task implementation plans
│   └── ISSUES.md                   [runtime] Blocker log
└── .opencode/
    ├── agents/
    │   ├── project-lead.md         Orchestrator (primary agent)
    │   ├── architect.md            Systems architect subagent
    │   ├── r-developer.md          Implementation engineer subagent
    │   └── tester.md               Test engineer subagent
    └── .gitignore                  [gitignored]
```

## Agent Pipeline

The workflow is a fixed linear pipeline driven by `project-lead`,
which gates every handoff behind user approval.

### Phase 1: Grill

`project-lead` interviews you relentlessly, one question at a
time, until every branch of the design tree is resolved. Coding
style questions are never asked — `.project/STYLE_GUIDE.md` is
treated as settled law.

### Phase 2: Objectives

The shared understanding is synthesized into
`.project/OBJECTIVES.md` (feature spec) and, if it does not yet
exist, `.project/ARCHITECTURE.md` (module-level design decisions).

### Phase 3: Architect

The `architect` subagent is dispatched to read OBJECTIVES.md and
write `.project/TASKS/<task>.md`. The output includes typed
function signatures, pseudocode, a file layout, data flow, and an
execution checklist split into `[IMPL]` and `[TEST]` items.

### Phase 4: R Developer

The `r-developer` subagent executes only `[IMPL]` checklist items.
It reads STYLE_GUIDE.md before writing any code, runs smoke tests
via `source()` on every modified file, and must never touch
`tests/testthat/`.

### Phase 5: Tester

The `tester` subagent writes `[TEST]` items, executes them with
`testthat::test_file()`, runs the full suite as a regression gate,
and classifies failures as test bugs (self-fixed, up to 3
attempts) or code bugs (escalated back to the R Developer). The
fix loop between R Developer and Tester runs up to 3 cycles. On
exhaustion, the deadlock is logged to `.project/ISSUES.md`.

### Domain Boundaries

- R Developer: reads test files but must never create or modify
  them
- Tester: reads implementation files but must never modify them
- Project Lead: the only agent that interacts with the user

## Skills

Skills are loaded from `~/.config/opencode/skills/` on demand when
their trigger phrases match.

| Skill | Role |
|---|---|
| `grill-me` | Interviews relentlessly until the plan is fully specified |
| `objectives` | Captures shared understanding into `.project/OBJECTIVES.md` |
| `code-implementation-plan` | Turns OBJECTIVES.md into `.project/TASKS/<task>.md` |
| `r-style` | Enforces R code conventions on all `.R`, `.qmd`, and `.Rmd` files |

> **Sync note:** `r-style` SKILL.md and `.project/STYLE_GUIDE.md`
> are near-duplicates. STYLE_GUIDE.md is the canonical source
> checked into this repo; `r-style` SKILL.md is the copy opencode
> loads at runtime. When conventions change, update both.

## Workspace Lifecycle

### Starting a New R Project

1. Copy this opencode config into the project (or keep this repo
   as a standalone setup and reference it).
2. Ensure skills are symlinked into `~/.config/opencode/skills/`.
3. Launch opencode in the project directory — `project-lead`
   activates automatically.

### Working Through a Feature

1. Describe the task to project-lead. It initiates the grill
   session.
2. After the grill, objectives are captured, then the architect
   writes an implementation plan.
3. On approval, the R Developer implements the code.
4. After implementation, the Tester writes and verifies tests.
5. Repeat for the next feature.

### Resetting After a Feature

- Delete or archive `.project/OBJECTIVES.md`.
- Clear `.project/TASKS/` of completed task files.
- `.project/ARCHITECTURE.md` persists across features — it
  accumulates module-level design decisions. The project-lead
  updates it after each task if new modules or structural changes
  were introduced.

## Modifying This Setup

### Adding a Skill

1. Create a `SKILL.md` in
   `~/my-skills/.config/opencode/skills/<name>/`.
2. Symlink it into `~/.config/opencode/skills/<name>`.
3. Reload opencode — skills are auto-discovered.

### Changing the Default Agent

Edit `opencode.jsonc` and change the `"default_agent"` value.

### Updating the Style Guide

1. Edit `.project/STYLE_GUIDE.md` (canonical).
2. Mirror the changes in
   `~/.config/opencode/skills/r-style/SKILL.md` (runtime copy).

### Adding an R Package

Add it to the Prerequisites section. Agents use
`package::function()` prefixes throughout, so no `library()`
calls are needed in any code produced by this setup.
