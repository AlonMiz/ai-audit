# AI Readiness Audit Report

**Project:** newz  
**Date:** 2026-05-14  
**Score:** 77/100 (77%) — Level 4: Optimized

---

## Detected Stack

Next.js 16, React 19, Drizzle ORM, Neon PostgreSQL, Vitest, Playwright

---

## Results

### 🔴 Critical [49/49 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T1-01 | Agent instruction file | ✅ | `AGENTS.md` found at root |
| T1-02 | Instruction file has commands | ✅ | `pnpm test:unit`, `pnpm test:e2e`, `pnpm validate` in AGENTS.md |
| T1-03 | Instruction file has conventions | ✅ | "Always prefer shadcn/ui", "NEVER use non-standard shell one-liners", "NEVER edit files via terminal" in AGENTS.md |
| T1-04 | README exists | ✅ | `README.md` found at root |
| T1-05 | README has quick start | ✅ | Prerequisites (Node.js 24+, pnpm 10+, Docker) + Quick Start with `pnpm install` / `pnpm dev` steps |
| T1-06 | Environment variable template | ✅ | `.env.example` found at root |
| T1-07 | Env vars documented | ✅ | `.env.example` has `#` comment blocks for every variable group |

### 🟡 Important [24/40 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T2-01 | Architecture doc | ✅ | `## Architecture` section in README.md lists framework, DB, ORM, API, auth, AI, testing |
| T2-02 | Tech stack documented | ✅ | README names Next.js 16 (framework), PostgreSQL/Neon (DB), Vitest + Playwright (testing) |
| T2-03 | Definition of done | ✅ | AGENTS.md: "A task is only complete when `pnpm validate` passes — no exceptions." |
| T2-04 | Single validation command | ✅ | `"validate"` script in `package.json` |
| T2-05 | Dev workflow skill | ⚠️ | `.agents/skills/` exists with 3 skills (neon-postgres, shadcn, ui-ux-pro-max) — none match `workflow`, `guidelines`, `dev`, `process`, `coding-standards` |
| T2-06 | Framework skill | ⚠️ | `.agents/skills/` has no skill matching `nextjs`, `next`, or `react` |
| T2-07 | Test commands documented | ✅ | AGENTS.md: `pnpm test:unit` (unit) and `pnpm test:e2e` (E2E) both documented |
| T2-08 | Pre-commit hooks configured | ⚠️ | `lefthook.yml` covers lint (`biome lint`) and format (`biome check`) but **no typecheck or test phase** |
| T2-09 | E2E tests configured | ✅ | `e2e/playwright.config.ts` + 7 spec files (feed, cluster, dismiss, search, settings, sidebar, smoke) |
| T2-10 | TDD workflow documented | ⚠️ | AGENTS.md and README contain no TDD/red-green-refactor language; only `task.prompt.md` (a prompt file, not instruction file) mentions it |

### 🟢 Advanced [4/11 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T3-01 | UI/Design skill | ✅ | `.agents/skills/ui-ux-pro-max` (matches `ui`, `ux`, `design`) and `.agents/skills/shadcn` (matches `shadcn`, `component`) |
| T3-02 | Brainstorming/planning skill | ❌ | No skill in `.agents/skills/` or `.claude/skills/` matching `brainstorm`, `planning`, `spec`, `writing-plans`, etc. |
| T3-03 | Debugging skill | ❌ | No skill matching `debug`, `diagnose`, `troubleshoot`, `systematic` in project skills folders |
| T3-04 | Database/ORM skill | ✅ | `.agents/skills/neon-postgres` name and description match `neon`, `postgres` |
| T3-05 | ADRs present | ❌ | No `docs/adr/` or `docs/decisions/` directory found |
| T3-06 | Domain/pipeline docs | ✅ | `src/server/pipeline/CLUSTERING.md` exists; also `docs/superpowers/` has many domain-specific plan docs |
| T3-07 | Subagents defined | ✅ | `.agents/` directory exists with `skills/` containing 3 skill definitions |
| T3-08 | Context scoped by path | ❌ | No AGENTS.md in subdirectory; all `.prompt.md` files either have `applyTo: "**"` or no `applyTo`; no `.instructions.md` or `.rules.md` in any subdirectory |
| T3-09 | Finishing/integration workflow | ❌ | No skill in project's `.agents/skills/` matching `finishing`, `merge`, `pr`, `pull-request`, `git-workflow`, `branch` (global skill referenced in AGENTS.md, but not installed locally) |
| T3-10 | MCP server configured | ❌ | No `.mcp.json` or MCP server block found in config files |
| T3-11 | Eval/drift config | ❌ | No eval dataset or drift monitoring config found |

---

## Recommendations

### [+4 pts] T2-05 — Dev workflow skill

Add a `dev-guidelines` skill to `.agents/skills/`. The AGENTS.md already references this skill by name but it's not installed in the project.

```bash
mkdir -p .agents/skills/dev-guidelines
```

`.agents/skills/dev-guidelines/SKILL.md`:
```markdown
---
name: dev-guidelines
description: Development workflow guidelines for this project — shadcn/ui-first components, terminal safety rules, E2E test requirements, and definition of done. Use when starting any implementation task, bug fix, or code change.
---

# Dev Guidelines

See AGENTS.md for the full set of project conventions.
```

### [+4 pts] T2-06 — Framework skill

Add a `nextjs-patterns` skill covering Next.js App Router, tRPC, and server/client component patterns.

```bash
mkdir -p .agents/skills/nextjs-patterns
```

`.agents/skills/nextjs-patterns/SKILL.md`:
```markdown
---
name: nextjs-patterns
description: Next.js App Router patterns for this project — server vs client components, tRPC usage, route handlers, data fetching, and React 19 best practices.
---

# Next.js Patterns
...
```

### [+4 pts] T2-08 — Pre-commit hooks: add typecheck phase

`lefthook.yml` covers lint and format but no typecheck. Add a typecheck command:

```yaml
pre-commit:
  commands:
    # ...existing commands...
    typecheck:
      glob: "*.{ts,tsx}"
      run: pnpm typecheck
```

### [+4 pts] T2-10 — TDD workflow documented

Add a TDD section to `AGENTS.md` or `README.md`. Example addition to AGENTS.md:

```markdown
## TDD Workflow

Follow red-green-refactor:
1. Write a failing test that describes the expected behaviour
2. Write the minimum code to make it pass
3. Refactor — clean up without changing behaviour
4. Run `pnpm validate` before marking done

Never write implementation code before a failing test exists.
```

### [+1 pt] T3-02 — Brainstorming/planning skill

Install or create a `brainstorming` or `writing-plans` skill locally:

```bash
mkdir -p .agents/skills/writing-plans
```

`.agents/skills/writing-plans/SKILL.md`:
```markdown
---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code — structures implementation plans as ordered tasks with test coverage notes.
---
```

### [+1 pt] T3-03 — Debugging skill

Add a `systematic-debugging` skill to the project:

```bash
mkdir -p .agents/skills/systematic-debugging
```

`.agents/skills/systematic-debugging/SKILL.md`:
```markdown
---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes — reproduce, minimise, hypothesise, instrument, fix, regression-test.
---
```

### [+1 pt] T3-05 — ADRs present

Create an ADR directory and record at least one architectural decision already made:

```bash
mkdir -p docs/adr
```

`docs/adr/0001-use-drizzle-orm.md`:
```markdown
# ADR 0001 — Use Drizzle ORM over Prisma

**Status:** Accepted  
**Date:** 2026-03-21

## Decision
Use Drizzle ORM with postgres-js driver for all database access.

## Rationale
Drizzle provides type-safe SQL with zero runtime overhead and works natively with Neon Serverless Postgres.
```

### [+1 pt] T3-08 — Context scoped by path

Add a path-scoped instruction file to a subdirectory, e.g. for the pipeline:

`src/server/pipeline/.instructions.md` (or use `applyTo` in a prompt):
```markdown
# Pipeline Instructions

These files implement the FeedFetcher and StoryProcessor. Key rules:
- Never import from `src/app/` — pipeline is server-only
- All AI calls must use the `ai` SDK, not direct provider SDKs
- See CLUSTERING.md for threshold rationale
```

Or add an `applyTo` frontmatter to a targeted prompt file (not `"**"`):
```yaml
---
applyTo: "src/server/pipeline/**"
---
```

### [+1 pt] T3-09 — Finishing/integration workflow

Install the `finishing-a-development-branch` skill locally (already referenced in AGENTS.md but not installed):

```bash
mkdir -p .agents/skills/finishing-a-development-branch
```

`.agents/skills/finishing-a-development-branch/SKILL.md`:
```markdown
---
name: finishing-a-development-branch
description: Use when implementation is complete and all tests pass — guides merge-to-main, pre-merge summary writing, and worktree cleanup.
---
```

---

*Generated by ai-audit — drop `.github/prompts/ai-audit.prompt.md` into any project to run this audit.*
