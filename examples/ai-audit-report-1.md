# AI Readiness Audit Report

**Project:** pecanify
**Date:** 2026-05-15
**Audit Version:** 0.0.3
**Score:** 84/100 (84%) — Level 4: Optimized

---

## Detected Stack

Next.js, React, TypeScript, Drizzle ORM, Neon Postgres, Vitest, Playwright, React Query, tRPC, Zod

---

## Results

### 🔴 Foundation [70/70 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T1-01 | Agent instruction file | ✅ | `AGENTS.md` found at root |
| T1-02 | Instruction file has commands | ✅ | `npm run dev`, `npm run test:unit:node`, `npm run lint:fix`, `npm run test:e2e` in AGENTS.md |
| T1-03 | Instruction file has conventions | ✅ | Arrow functions, no IIFEs, naming conventions (kebab-case, PascalCase, use prefix), no inline import() types |
| T1-04 | README exists | ✅ | `README.md` found at root |
| T1-05 | README has quick start | ✅ | "Contributing - running local" section with prerequisites (Docker, npm) and steps (npm i, copy .env, docker, npm run dev) |
| T1-06 | Environment variable template | ✅ | `.env.example` found at root |
| T1-07 | Env vars documented | ✅ | Section comments: `######## DATABASE ########`, `# Clerk`, `# AWS IAM`, `# Upstash`, etc. |
| T1-08 | Instruction file consistency | N/A | Only 1 instruction file found (AGENTS.md) |
| T1-09 | Definition of done | ✅ | "Mandatory Verification" section: `get_errors`, `npm run test:unit:node`, `npm run lint:fix`; "Never mark a task as completed without running all three steps" |
| T1-10 | Single validation command | ✅ | `"validate": "npm run type-check && npm run lint && npm run knip"` in package.json |
| T1-11 | Dev workflow skill | ✅ | `using-superpowers` skill: "Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response" |

### 🟡 Important [40/56 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T2-01 | Architecture doc | ✅ | AGENTS.md contains "Core Technologies & Architecture" and "Database Schema Architecture" sections with entity/table descriptions and directory structure |
| T2-02 | Tech stack documented | ✅ | Framework: "Next.js 16 with App Router + TypeScript"; DB: "PostgreSQL with Drizzle ORM"; Testing: Vitest + Playwright named in AGENTS.md |
| T2-06 | Framework skill | ⚠️ | Skills folder exists but no Next.js or React framework skill in `.agents/skills/`. Detected frameworks: Next.js, React |
| T2-07 | Test commands documented | ✅ | Unit: `npm run test:unit:node`; E2E: `npm run test:e2e`; Storybook: `npm run storybook:test` — all in AGENTS.md |
| T2-08 | Pre-commit hooks configured | ⚠️ | `lefthook.yml` covers Phase 1 (lint) and Phase 2 (format) via biome, but Phase 3 (typecheck or test) is absent |
| T2-09 | E2E tests configured | ✅ | `playwright.config.ts` at root; 15+ `*.spec.ts` files in `tests/` |
| T2-10 | TDD workflow (informational) | ℹ️ | Present — `test-driven-development` and `red-green-refactor` skills exist; AGENTS.md: "NEVER just change the test to match the current implementation" |
| T2-11 | Operational guardrails documented | ✅ | 3 signals found: (1) prohibition language (NEVER git push, NEVER use as, NEVER use @ts-ignore), (2) shell one-liner ban ("Never inline bash loops in ad-hoc terminal calls"), (3) script discipline ("create a reusable script in scripts/") |
| T2-12 | Planning protocol documented | ✅ | When: "Always create a plan before starting a complex task"; Where: "Store plans in `/plans/` directory"; How: `writing-plans` skill referenced |
| T2-13 | Test strategy documented | ⚠️ | 4 test types named (Unit, E2E, Storybook, API) but no explicit when-to-use selection guidance — only procedural "how to run" info |
| T2-14 | Branch finishing protocol documented | ⚠️ | Only 1 signal: "NEVER git push without explicit permission from the developer. Commit locally, then wait to be told to push." Missing: pre-merge summary, cleanup protocol |
| T2-15 | Code quality tools configured | ✅ | `knip` v6.0.2 in devDependencies; `"knip": "knip --config infra/knip.json"` script; also runs in `validate` script |
| T2-16 | High-impact library skills | ⚠️ | Skills folder exists but no skill for React Query, tRPC, or Drizzle usage patterns. All three detected in package.json |
| T2-17 | Language discipline documented | ✅ | (1) no any: "NO 'any', 'as', or '@ts-ignore/expect-error'"; (2) no as: "NEVER USE 'as' WITHOUT EXPLICIT PERMISSION"; (3) strict: "Use TypeScript strict mode" |
| T2-18 | Schema validation configured | ✅ | `zod` v4.2.1 in dependencies; AGENTS.md: "Use Zod schemas for all input validation", "Validate inputs with Zod schemas" |
| T2-19 | No duplicate instruction content | N/A | Only 1 instruction file found (AGENTS.md) |
| T2-20 | Code collaboration protocol documented | ✅ | 2/3 signals: (1) commit conventions: absent; (2) PR/integration via `finishing-a-development-branch` skill reference; (3) code review via `requesting-code-review` and `receiving-code-review` skill references |

### 🟢 Advanced [7/11 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T3-01 | UI/Design skill | ✅ | `create-storybook-story`: "Create Storybook stories with interaction tests" — covers UI component library documentation |
| T3-02 | Brainstorming/planning skill | ✅ | `brainstorming`: "You MUST use this before any creative work — creating features, building components..." |
| T3-03 | Debugging skill | ✅ | `systematic-debugging`: "Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes" |
| T3-04 | Database/ORM skill | ✅ | `modify-db-schema`: "Modify database schema safely with migrations and plans. Use this when asked to 'add column', 'change table', 'update db'" |
| T3-05 | ADRs present | ❌ | No `docs/adr/` or `docs/decisions/` directory found |
| T3-06 | Domain/pipeline docs | ✅ | `docs/superpowers/plans/2026-04-06-gtm-conversion-tracking.md`; `plans/casa-saq-2026.md`, `plans/casa-security-audit-2026.md`, `plans/inforu-whatsapp-template-message.md` |
| T3-07 | Subagents defined | ✅ | `.agents/prompts/debug-production-error.prompt.md` and `merge-from-main.prompt.md` both have `tools:` in YAML frontmatter restricting agent capabilities |
| T3-08 | Context scoped by path | ❌ | No AGENTS.md, .instructions.md, or .rules.md in subdirectories; no `.prompt.md` with `applyTo:` targeting specific paths |
| T3-09 | Finishing/integration workflow | ✅ | `finishing-a-development-branch`: "guides completion of development work by presenting structured options for merge, PR, or cleanup" |
| T3-10 | MCP server configured | ❌ | No `.vscode/mcp.json` or `mcp.json` found. `infra/mcp/` contains an N8N test workflow, not an MCP server config |
| T3-11 | Eval/drift config | ❌ | No `evals/` or `eval/` directory found |

---

## Recommendations

### [+4 pts] T2-06 — Framework skill missing

In `.agents/skills/`, create `nextjs-patterns/SKILL.md` with frontmatter `name: nextjs-patterns` and a description covering Next.js App Router conventions, server/client component boundaries, data fetching patterns (server actions vs tRPC), route handlers, and Mantine-specific layout patterns used in this project.

---

### [+4 pts] T2-08 — Pre-commit hooks missing typecheck phase

In `lefthook.yml`, add a `pre-push` command (or extend the existing one) that runs `npm run type-check` to cover Phase 3 (typecheck). The current config runs `biome check` for lint/format but never runs the TypeScript compiler, so type errors can be pushed without being caught locally.

---

### [+4 pts] T2-13 — Test strategy lacks selection guidance

In `AGENTS.md`, under "Testing Patterns", add explicit when-to-use guidance. For example: use **unit tests** for service logic, data transformations, and business rules in isolation; use **Storybook** for interactive component behavior and visual states; use **E2E** for critical user flows (auth, card creation, invoicing); use **API tests** for tRPC procedure contracts and error handling.

---

### [+4 pts] T2-14 — Branch finishing protocol incomplete

In `AGENTS.md`, add a "Branch Finishing" section covering: (1) pre-merge summary — what to write in the PR description (changed files, rationale, test coverage); (2) cleanup — delete the feature branch after merge, remove worktrees if using git worktrees. Currently only "NEVER git push without permission" is documented; the rest of the finishing workflow lives in the skill but not in AGENTS.md itself.

---

### [+4 pts] T2-16 — High-impact library skills missing

Create three skill stubs in `.agents/skills/`:

1. `trpc-patterns/SKILL.md` — `name: trpc-patterns`, description covering `protectedProcedure`, input validation, error handling with `TRPCError`, and router composition patterns used in this project.

2. `react-query-patterns/SKILL.md` — `name: react-query-patterns`, description covering query key conventions, optimistic updates, `useMutation` with `onSuccess` invalidation, and pagination patterns using `@tanstack/react-query` v5.

3. `drizzle-patterns/SKILL.md` — `name: drizzle-patterns`, description covering transaction patterns, `orgId` filtering for multi-tenancy, migration workflow (`db:generate-migration` → `db:migrate:dev`), and type-safe query patterns with `drizzle-orm`.

---

### [+1 pt] T3-05 — No ADRs

Create `docs/adr/` and add at least one ADR documenting a key architectural decision (e.g., choice of tRPC over REST, Drizzle over Prisma, Clerk for multi-tenant auth, or the Cards/CardTypes/Fields dynamic entity model). Use a standard ADR format: Context, Decision, Consequences.

---

### [+1 pt] T3-08 — No path-scoped context

Add `applyTo:` frontmatter to one or more `.agents/prompts/*.prompt.md` files to scope them to relevant paths (e.g., `applyTo: "src/server/**"` for backend-focused prompts, `applyTo: "tests/**"` for test prompts). This signals to agents which prompts are relevant for which parts of the codebase.

---

### [+1 pt] T3-10 — No MCP server configured

Create `.vscode/mcp.json` to register at least one MCP server (e.g., the BetterStack server already referenced in `debug-production-error.prompt.md`, or a database introspection server for Drizzle). Minimum config: `{ "servers": { "<name>": { "command": "...", "args": [...] } } }`.

---

### [+1 pt] T3-11 — No eval/drift config

Create an `evals/` directory with at least a seed file defining prompt evaluation scenarios — e.g., sample prompts that an agent should handle correctly, with expected behaviors. This lets you detect when changes to AGENTS.md or skills degrade agent performance.

---

*Generated by [ai-audit](https://github.com/AlonMiz/ai-audit) v2.0 — install with `npx skills@latest add AlonMiz/ai-audit`*
