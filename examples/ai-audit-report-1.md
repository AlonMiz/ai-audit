# AI Readiness Audit Report

**Project:** pecanify
**Date:** May 15, 2026
**Score:** 85/100 (85%) — Level 4: Optimized

---

## Detected Stack

Next.js 16, React 19, TypeScript, Drizzle ORM, Neon Postgres + pg, Vitest, Playwright, Storybook, Mantine UI, tRPC, Clerk Auth

---

## Results

### 🔴 Critical [49/49 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T1-01 | Agent instruction file | ✅ | `AGENTS.md` found at root |
| T1-02 | Instruction file has commands | ✅ | `npm run test:unit:node`, `npm run lint:fix`, `npm run dev`, `npm run docker:start` etc. in AGENTS.md |
| T1-03 | Instruction file has conventions | ✅ | Naming conventions, arrow-function rule, no `any`/`as`, no inline `import()` types, SCREAMING_SNAKE_CASE constants, file/component naming in AGENTS.md |
| T1-04 | README exists | ✅ | `README.md` found at root |
| T1-05 | README has quick start | ✅ | Prerequisites (npm, Docker) and run steps (`npm i`, `.env.example` copy, `npm run db:run:local`, `npm run dev`) documented |
| T1-06 | Environment variable template | ✅ | `.env.example` found at root |
| T1-07 | Env vars documented | ✅ | Section comments (`######## DATABASE ########`, `# Clerk`, `# AWS IAM`, `# Upstash`, etc.) throughout `.env.example` |

### 🟡 Important [44/56 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T2-01 | Architecture doc | ✅ | AGENTS.md has "Database Schema Architecture" section describing Cards System, Activities, Relations, Permissions, Finance + Directory Structure |
| T2-02 | Tech stack documented | ✅ | AGENTS.md names framework (Next.js 16, TypeScript), database (PostgreSQL + Drizzle ORM), and test tools (Vitest, Playwright) |
| T2-03 | Definition of done | ✅ | "Mandatory Verification" section in AGENTS.md: run `get_errors`, run tests, run `npm run lint:fix` — "Never mark a task as completed without running all three steps." |
| T2-04 | Single validation command | ✅ | `package.json` `scripts.validate`: `"npm run type-check && npm run lint && npm run knip"` |
| T2-05 | Dev workflow skill | ✅ | `using-superpowers`: "Use when starting any conversation — establishes how to find and use skills, requiring Skill tool invocation before ANY response" |
| T2-06 | Framework skill | ❌ | No skill name, description, or directory contains `next`, `nextjs`, or `react` — framework-specific skill absent |
| T2-07 | Test commands documented | ✅ | AGENTS.md lists `npm run test:unit:node` (unit) and `npm run test:e2e` (E2E) with explicit notes |
| T2-08 | Pre-commit hooks configured | ⚠️ | `lefthook.yml` found; Biome covers Phase 1 (lint) and Phase 2 (format) on pre-commit and pre-push; Phase 3 (typecheck or test) missing |
| T2-09 | E2E tests configured | ✅ | `playwright.config.ts` at root; 15+ spec files in `tests/` (e.g. `app.spec.ts`, `card.spec.ts`, `auth.setup.ts`) |
| T2-10 | TDD workflow documented | ✅ | Skill `red-green-refactor` frontmatter: "Always follow Red → Green → Refactor order. Never write a passing test first." Referenced in AGENTS.md skills table |
| T2-11 | Operational guardrails documented | ✅ | 3 signals found: (1) `NEVER git push`, `NEVER USE 'as'`, `NEVER USE '@ts-ignore'`; (2) "Never inline bash loops in ad-hoc terminal calls"; (4) "create a reusable script in `scripts/` and register it as an `npm run` script" |
| T2-12 | Planning protocol documented | ✅ | AGENTS.md: When — "Always create a plan before starting"; Where — "Store plans in `/plans/` directory"; How — `writing-plans` skill listed in skills table |
| T2-13 | Test strategy documented | ⚠️ | 4 test types named (Unit/Vitest, E2E/Playwright, Storybook, API); no selection guidance on when to use each type over another |
| T2-14 | Branch finishing protocol documented | ✅ | AGENTS.md skills table references `finishing-a-development-branch`: "decide how to integrate (merge/PR/cleanup)" covering signals 1 (merge strategy) and 3 (cleanup) |

### 🔍 Flags (non-scoring)

| Flag | Result | Note |
|------|--------|------|
| DIV | ℹ️ No divergence | Single instruction file found: `AGENTS.md`. No `CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`, or `.instructions.md` present. |

### 🟢 Advanced [6/11 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T3-01 | UI/Design skill | ✅ | `create-storybook-story`: "Create Storybook stories with interaction tests following project guidelines" — covers component UI documentation |
| T3-02 | Brainstorming/planning skill | ✅ | `brainstorming`: "Explores user intent, requirements and design before implementation" |
| T3-03 | Debugging skill | ✅ | `systematic-debugging`: "Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes" |
| T3-04 | Database/ORM skill | ✅ | `modify-db-schema`: "Modify database schema safely with migrations and plans" — covers Drizzle/DB operations |
| T3-05 | ADRs present | ❌ | No `docs/adr/` or `docs/decisions/` directory found |
| T3-06 | Domain/pipeline docs | ✅ | `db/readme.md` documents Drizzle ORM domain patterns; `src/app/docs/` has domain MDX docs (roles-and-permissions, automations, ai-assistant, etc.) |
| T3-07 | Subagents defined | ❌ | No `.agents/agents/` or `.claude/agents/` directory found; `.agents/` has only `skills/` and `prompts/` |
| T3-08 | Context scoped by path | ❌ | No AGENTS.md in subdirectories; prompt files in `.agents/prompts/` lack `applyTo:` path-scoping frontmatter |
| T3-09 | Finishing/integration workflow | ✅ | `finishing-a-development-branch`: "guides completion of development work by presenting structured options for merge, PR, or cleanup" |
| T3-10 | MCP server configured | ❌ | No `.vscode/mcp.json` or `mcp.json` found |
| T3-11 | Eval/drift config | ❌ | No `evals/` or `eval/` directory found |

---

## Recommendations

### [+4 pts] T2-06 — Framework skill (Next.js/React)

Create `.agents/skills/nextjs-patterns/SKILL.md` with:
- Frontmatter: `name: nextjs-patterns`, `description: Next.js App Router and React patterns for this project — server components, tRPC integration, Mantine UI, Clerk auth, and file-based routing conventions. Use when building new pages, API routes, or components.`
- Content: Next.js App Router conventions, when to use server vs client components, how tRPC routes are structured, Clerk auth patterns, and Mantine component usage patterns from this codebase.

---

### [+4 pts] T2-08 — Pre-commit hooks: missing typecheck phase

In `lefthook.yml`, add a `pre-push` command (or extend the existing one) that runs `npm run type-check` after the Biome check, so type errors are caught before pushing. This covers Phase 3 (typecheck). Example addition:

```yaml
pre-push:
  commands:
    check:
      glob: "*.{js,ts,cjs,mjs,d.cts,d.mts,jsx,tsx,json,jsonc}"
      run: npx @biomejs/biome check --no-errors-on-unmatched --files-ignore-unknown=true --diagnostic-level=error --colors=off {push_files}
    typecheck:
      run: npm run type-check
```

---

### [+4 pts] T2-13 — Test strategy: missing selection guidance

In `AGENTS.md`, under the "Testing Patterns" section, add a test selection guide explaining when to use each test type. Recommended language:
- **Unit (Vitest)**: business logic, service functions, utilities, and tRPC procedure input/output validation — fast, no browser
- **Storybook**: isolated UI component rendering, visual states, and component interaction
- **E2E (Playwright)**: full user flows, multi-step interactions, and cross-page navigation — requires running dev server
- **API tests**: tRPC procedure authentication, schema validation, and error scenarios

---

### [+1 pt] T3-05 — ADRs

Create a `docs/adr/` directory with at least one Architecture Decision Record. Start with the most impactful past decisions, e.g. `docs/adr/001-trpc-over-rest.md`, `docs/adr/002-drizzle-over-prisma.md`, or `docs/adr/003-clerk-multi-tenancy.md`. Standard ADR format: Title, Status, Context, Decision, Consequences.

---

### [+1 pt] T3-07 — Subagents defined

Create at least one subagent file with its own tool restrictions or model config. Example: create `.agents/agents/code-reviewer.md` with frontmatter:
```yaml
---
name: code-reviewer
description: Read-only code review subagent. Reviews diffs and files for correctness, security, and convention compliance.
tools: [read, search]
model: claude-opus-4
---
```

---

### [+1 pt] T3-08 — Context scoped by path

Add path-scoped context to high-complexity subdirectories. For example, create `.agents/prompts/db-changes.prompt.md` with `applyTo: db/**` frontmatter covering migration conventions, or add an `AGENTS.md` inside `src/server/` describing the tRPC router + service layer architecture for that subtree.

---

### [+1 pt] T3-10 — MCP server configured

Create `.vscode/mcp.json` with at least one MCP server configured — e.g. the GitHub MCP for PR/issue management, or a BetterStack MCP for log querying (referenced in `debug-production-error.prompt.md`).

---

### [+1 pt] T3-11 — Eval/drift config

Create an `evals/` directory with at least one evaluation dataset or script that tests whether agent instructions remain effective. A minimal setup: a `evals/README.md` describing the eval strategy, plus one `evals/smoke/` folder with a sample task + expected output for regression checking.

---

*Generated by [ai-audit](https://github.com/AlonMiz/ai-audit) — install with `npx skills@latest add AlonMiz/ai-audit`*
