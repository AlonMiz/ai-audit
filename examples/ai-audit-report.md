# AI Readiness Audit Report

**Project:** newz  
**Date:** 2026-05-15  
**Score:** 85/116 (73%) — Level 3: Standardized

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

### 🟡 Important [32/56 pts]

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
| T2-11 | Operational guardrails documented | ✅ | AGENTS.md has `NEVER use non-standard shell one-liners`, `NEVER edit files via terminal`, `ALWAYS ask before running package installs`, `pnpm <script>` discipline, `./tmp/` for temp files — 5/6 signals |
| T2-12 | Planning protocol documented | ⚠️ | AGENTS.md has no "create a plan before..." instruction; `/plans/` directory exists but storage location not mandated in instructions; no planning skill referenced |
| T2-13 | Test strategy documented | ⚠️ | AGENTS.md lists `pnpm test:unit` and `pnpm test:e2e` but provides no guidance on when to choose unit vs integration vs E2E |
| T2-14 | Branch finishing protocol documented | ✅ | AGENTS.md has merge strategy (merge to `main` locally), pre-merge summary requirement, and worktree cleanup instruction — 3/3 signals |

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

Create `.agents/skills/dev-guidelines/SKILL.md` with `name: dev-guidelines` and a description referencing this project's conventions — shadcn/ui-first, terminal safety rules, E2E requirements, and definition of done.

### [+4 pts] T2-06 — Framework skill

Create `.agents/skills/nextjs-patterns/SKILL.md` with `name: nextjs-patterns` and a description covering Next.js App Router patterns, server vs client components, tRPC usage, and React 19 best practices for this project.

### [+4 pts] T2-08 — Pre-commit hooks: add typecheck phase

In `lefthook.yml`, add a `typecheck` command to the `pre-commit` hook that runs `pnpm typecheck` on `*.{ts,tsx}` files.

### [+4 pts] T2-10 — TDD workflow documented

In `AGENTS.md`, add a "TDD Workflow" section specifying red-green-refactor: write a failing test first, write minimum code to pass, refactor, then run `pnpm validate`.

### [+4 pts] T2-12 — Planning protocol documented

In `AGENTS.md`, add a "Planning" section specifying: (1) create a plan before starting any complex task, (2) store plans in `/plans/` as markdown files with task checklists, (3) invoke the `writing-plans` skill for structured plans.

### [+4 pts] T2-13 — Test strategy documented

In `AGENTS.md`, add a "Test Strategy" section explaining when to use each test type: unit tests (Vitest) for pure logic and services, Playwright E2E for user-facing flows and pages — and that every implementation plan must include a test coverage decision.

### [+1 pt] T3-02 — Brainstorming/planning skill

Create `.agents/skills/writing-plans/SKILL.md` with `name: writing-plans` and a description for use when structuring implementation plans before writing code.

### [+1 pt] T3-03 — Debugging skill

Create `.agents/skills/systematic-debugging/SKILL.md` with `name: systematic-debugging` and a description for use when encountering bugs or test failures — reproduce, minimise, hypothesise, fix, regression-test.

### [+1 pt] T3-05 — ADRs present

Create `docs/adr/` and add at least one ADR recording an architectural decision already made (e.g. choosing Drizzle ORM, Neon Postgres, or the clustering threshold approach).

### [+1 pt] T3-08 — Context scoped by path

Add an `applyTo` frontmatter to a targeted `.prompt.md` in `src/server/pipeline/` scoped to `src/server/pipeline/**` instead of `"**"` — or add a `.instructions.md` in that subdirectory with pipeline-specific rules.

### [+1 pt] T3-09 — Finishing/integration workflow

Create `.agents/skills/finishing-a-development-branch/SKILL.md` with `name: finishing-a-development-branch` and a description for use when implementation is complete — guides pre-merge summary, merge to main, and worktree cleanup. (Already referenced in AGENTS.md but not installed locally.)

---

*Generated by [ai-audit](https://github.com/AlonMiz/ai-audit) — install with `npx skills@latest add AlonMiz/ai-audit`*
