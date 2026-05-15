---
name: ai-audit
description: Audit any project's AI readiness. Scores 32 checks across 3 tiers, produces a 5-level maturity rating, and writes ai-audit-report.md with concrete fix recommendations. Run once on a new project or after major changes.
---

You are running an **AI Readiness Audit** on this codebase. Your job is to evaluate how well the project is set up for AI agents to work in it effectively.

**Rules:**
- Read files only. Do NOT modify any project file except writing `ai-audit-report.md` at the end.
- Be systematic: work through every check in order before scoring.
- Be evidence-based: record exactly what you found (filename, line content) for each check.
- Do not infer intent — if a file is absent, mark it absent.

---

## STEP 1 — READ FILES IN PRIORITY ORDER

Read these files before running any checks. If context limits prevent reading a file, note it and mark dependent checks as ⚠️ "file not read — context budget".

**Priority order:**
1. Root level: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`, `.instructions.md`, `README.md`, `package.json`, `.env.example`, `.env.template`, `.env.sample`, `.env.dist`, `lefthook.yml`, `.lefthook.yml`, `.pre-commit-config.yaml`
2. Skills folders: `.agents/skills/` and `.claude/skills/` — list all skill directories, then open each skill's `SKILL.md` or `README.md` to read its YAML frontmatter (`name:`, `description:`)
3. Docs: `docs/` folder — list all `.md` files, read filenames and first-level headings
4. E2E: `e2e/` or `tests/` — list files only (no need to read content)
5. Config files: `playwright.config.*`, `cypress.config.*`, `Makefile`, `Justfile`

**Never read `src/` source code files** — they are not needed for any check.

---

## STEP 2 — DETECT STACK

Parse `package.json` `dependencies` and `devDependencies`. Match against this table:

| Dependency | Framework tag | Expected skill keyword |
|------------|--------------|------------------------|
| `next` | Next.js | `nextjs`, `next`, `react` |
| `react` (without `next`) | React | `react` |
| `vue` | Vue | `vue` |
| `svelte` | Svelte | `svelte` |
| `@angular/core` | Angular | `angular` |
| `express`, `fastify`, `hono`, `koa` | Node API | `node`, `express`, `api` |
| `drizzle-orm` | Drizzle | `drizzle`, `database`, `orm` |
| `prisma` | Prisma | `prisma`, `database` |
| `typeorm` | TypeORM | `typeorm`, `database` |
| `@neondatabase/serverless` or any `@neondatabase/*` | Neon Postgres | `neon`, `postgres`, `database` |
| `pg`, `postgres` | Postgres | `postgres`, `database` |
| `vitest` | Vitest | `vitest`, `test` |
| `jest` | Jest | `jest`, `test` |
| `@playwright/test` or `playwright` | Playwright | `playwright`, `e2e` |
| `cypress` | Cypress | `cypress`, `e2e` |

If `package.json` is absent: mark stack as "unknown" and skip T2-06.

**Keyword matching rule for all skill-based checks:** Match keywords against (1) the skill's YAML frontmatter `description:` field, (2) the `name:` field, or (3) the skill filename/directory name. Full-text content search is NOT required. You must open each skill file to read frontmatter — filename alone is not sufficient.

---

## STEP 3 — RUN ALL 32 CHECKS

Work through every check. For each one record: status (✅ / ❌ / ⚠️), points earned (✅ = full points, ⚠️ = 0, ❌ = 0), and a brief evidence note.

### Status symbols
- ✅ Pass — criteria fully met
- ❌ Fail — criteria not met
- ⚠️ Partial — file exists but content check failed (scores 0; gets a "you have the file, here's what to add" recommendation)

---

### TIER 1 — CRITICAL (7 pts each, 49 pts max)

**T1-01 — Agent instruction file exists** [7 pts]
Look for any of: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`, `.instructions.md` at the root.
- ✅ if any found | ❌ if none found
- Partial: N/A (binary)

**🔍 DIV — Instruction file divergence** [0 pts — informational flag only]
If T1-01 found **2 or more** instruction files, read all of them and check for contradictions: conflicting commands, duplicate content, or rules that contradict each other.
- Record which files were found
- If divergence detected: add a ⚠️ NOTE in the report under a "Flags" section
- If consistent (or only 1 file): note "No divergence detected"

**T1-02 — Instruction file has run commands** [7 pts]
Read the instruction file found in T1-01. Look for test, build, lint, validate, or dev commands (e.g., `pnpm test`, `npm run build`, `pytest`, `make test`).
- ✅ if at least one command is present
- ⚠️ if instruction file exists (T1-01 ✅) but no commands found
- ❌ if no instruction file exists

**T1-03 — Instruction file has coding conventions** [7 pts]
Read the instruction file. Look for at least one rule about code style, naming, tooling preference, or prohibited patterns (e.g., "use camelCase", "never use var", "prefer named exports").
- ✅ if at least one convention rule found
- ⚠️ if instruction file exists but no convention rules found
- ❌ if no instruction file

**T1-04 — README exists** [7 pts]
Look for `README.md` at root.
- ✅ if found | ❌ if absent
- Partial: N/A (binary)

**T1-05 — README has quick start** [7 pts]
Read `README.md`. Look for a section that tells a new developer how to get the project running locally — prerequisites (runtime versions, tools) AND install/run steps.
- ✅ if both prerequisites and run steps are present
- ❌ if README is absent or neither section is present
- Note: README without a quick start is still a ❌ (not ⚠️) — the README is not a "present file with missing content" for this check

**T1-06 — Environment variable template** [7 pts]
Look for any of: `.env.example`, `.env.template`, `.env.sample`, `.env.dist` at root.
- ✅ if any found | ❌ if none found
- Partial: N/A (binary)

**T1-07 — Env vars are documented** [7 pts]
Read the env template file found in T1-06. Look for: at least one line starting with `#` that describes a variable's purpose, OR a README-style description block above variables.
- ✅ if documentation comments present
- ⚠️ if template exists (T1-06 ✅) but no comments/descriptions found
- ❌ if no template file

---

### TIER 2 — IMPORTANT (4 pts each, 44 pts max)

**T2-01 — Architecture or context doc** [4 pts]
Look for any documentation that describes how the system is structured — could be a `CONTEXT.md`, `ARCHITECTURE.md`, a `docs/adr/` directory, or any `.md` file in `docs/` whose name or top-level heading indicates it describes the system design, data flow, services, schema, or domain.
- ✅ if any match | ❌ if none found

**T2-02 — Tech stack documented** [4 pts]
Read `README.md` and the instruction file. Look for explicit naming of: the primary framework/language, database, and test tool.
- ✅ if all three categories (framework, DB, testing) are named
- ❌ if README and instruction file are absent or mention fewer than two stack components
- Note: "TypeScript + Next.js + Postgres + Vitest" = ✅; "TypeScript project" alone = ❌

**T2-03 — Definition of done documented** [4 pts]
Read the instruction file and README. Look for explicit "definition of done" language, OR a description of what "done" means (e.g., "a task is complete when all tests pass", "pnpm validate must pass").
- ✅ if clear definition found
- ⚠️ if instruction file exists but no done-definition language found
- ❌ if no instruction file

**T2-04 — Single validation command** [4 pts]
Check `package.json` scripts for a key named `validate`, `check`, `ci`, or `test:all` — OR check `Makefile`/`Justfile` for a line matching `^(validate|check|ci|test):`.
- ✅ if found | ❌ if not found
- Partial: N/A (binary)

**T2-05 — Dev workflow skill** [4 pts]
List `.agents/skills/` or `.claude/skills/`. Look for a skill whose name or description indicates it covers development workflow, coding guidelines, or project conventions — the kind of skill an agent should load at the start of every task.
- ✅ if a matching skill found
- ⚠️ if skills folder exists but no matching skill found
- ❌ if no skills folder

**T2-06 — Framework skill present** [4 pts]
Using the expected skill keyword(s) from the detected stack (Step 2), check each skill's frontmatter `name:`, `description:`, and directory name for a match.
- ✅ if a matching skill found
- ⚠️ if skills folder exists but no framework skill matches
- ❌ if no skills folder or stack is unknown
- Skip (mark N/A, award 0) if `package.json` is absent

**T2-07 — Test commands documented** [4 pts]
Read the instruction file and README. Look for explicit commands for both: (1) unit/integration tests AND (2) E2E/browser tests. The commands should be runnable as-is (e.g. `pnpm test:unit`, `pytest`, `make test`, `npx playwright test`).
- ✅ if both unit and E2E commands documented | ❌ otherwise

**T2-08 — Pre-commit hooks configured** [4 pts]
Look for a pre-commit hook config (`lefthook.yml`, `.lefthook.yml`, `.husky/`, `.pre-commit-config.yaml`). If found, read it and assess which quality phases are covered:
- Phase 1 — Lint: checks code style/quality violations
- Phase 2 — Format: enforces consistent code formatting
- Phase 3 — Typecheck or test: catches type errors or runs a fast test suite

Scoring:
- ✅ all three phases covered
- ⚠️ config exists, 1–2 phases covered
- ❌ config absent OR config exists but covers none of the three phases

**T2-09 — E2E tests configured** [4 pts]
Look for: `playwright.config.ts`, `playwright.config.js`, `cypress.config.ts`, `cypress.config.js`, `nightwatch.conf.js`, `wdio.conf.js`. If found, also verify at least one `*.spec.*` or `*.e2e.*` file exists in `e2e/` or `tests/`.
- ✅ if config AND at least one spec file found | ❌ otherwise
- Partial: N/A (binary)

**T2-10 — TDD workflow documented** [4 pts]
Read the instruction file, README, and skill files' frontmatter. Look for language describing a test-first development process — writing a failing test before writing implementation code, the red-green-refactor loop, or equivalent.
- ✅ if TDD process described
- ⚠️ if instruction file exists but no TDD language found
- ❌ if no instruction file

**T2-11 — Operational guardrails documented** [4 pts]
Read the instruction file. Look for evidence that agent operational safety has been addressed. Check for at least 3 of these 6 concerns:
1. **Prohibition language** — explicit `NEVER` / `do not` rules applied to terminal commands or tool usage
2. **Shell one-liner ban** — prohibition of ad-hoc shell scripts or complex one-liners (sed, awk, piping, inline node/python, etc.)
3. **No terminal file editing** — rule that files must be edited with file-editing tools, not via terminal commands
4. **Script discipline** — instruction to wrap operations in named package.json scripts rather than running raw commands
5. **Install gate** — requirement to ask or confirm before installing packages
6. **Temp file safety** — instruction on where to write temporary files (inside the workspace, not system paths)

Scoring:
- ✅ if 3 or more signals found
- ⚠️ if 1–2 signals found — recommendation lists the missing signals
- ⚠️ if 0 signals found and instruction file exists — recommendation lists all 6
- ❌ if no instruction file

**T2-12 — Planning protocol documented** [4 pts]
Read the instruction file. Look for ALL THREE of:
1. **When** — language like `before starting`, `before implementing`, `before complex tasks`, `create a plan`, `always plan`
2. **Where** — an explicit storage path like `/plans/`, `docs/plans/`, `plans/`, or any named directory for plans
3. **How** — reference to a planning skill, spec approach, or TDD/design step

Scoring:
- ✅ if 2 or more of the three signals present
- ⚠️ if 1 signal present — recommendation tells the agent which parts are missing and what to add
- ⚠️ if none present and instruction file exists — recommendation provides a default planning protocol to add
- ❌ if no instruction file

**T2-13 — Test strategy documented** [4 pts]
Read the instruction file and README. Look for:
1. **Multiple test types named** — at least 2 distinct test types mentioned (e.g. unit, integration, component, Storybook, E2E, browser tests)
2. **Selection guidance** — any language that helps an agent decide which test type to use for a given task. This includes explicit rules like "unit for logic, E2E for user flows", comparative guidance, scope-based guidance, or a description of what each test type is for.

Scoring:
- ✅ both present — test types listed AND guidance on when to use which
- ⚠️ if test types listed but no selection guidance — recommendation suggests adding a test selection guide
- ⚠️ if neither present and instruction file exists — recommendation provides a default test strategy section to add
- ❌ if no instruction file

**T2-14 — Branch finishing protocol documented** [4 pts]
Read the instruction file. Look for at least 2 of these 3 concerns addressed:
1. **Merge/integration strategy** — how completed work gets integrated (merge to main, open a PR, squash, etc.)
2. **Pre-merge summary** — requirement to write a summary of what was done before merging
3. **Cleanup** — what to do after merging (remove worktree, delete branch, etc.)

Scoring:
- ✅ 2 or more signals found
- ⚠️ 0–1 signals and instruction file exists — recommendation provides a default finishing protocol to add
- ❌ if no instruction file

---

### TIER 3 — ADVANCED (1 pt each, 11 pts max)

**T3-01 — UI/Design skill** [1 pt]
Look for a skill whose name or description indicates it covers UI, UX, design systems, component libraries, or styling.
- ✅ if found | ❌ if not found

**T3-02 — Brainstorming/planning skill** [1 pt]
Look for a skill whose name or description indicates it covers brainstorming, ideation, writing specs, or planning tasks before implementation.
- ✅ if found | ❌ if not found

**T3-03 — Debugging skill** [1 pt]
Look for a skill whose name or description indicates it covers debugging, diagnosing bugs, or systematic troubleshooting.
- ✅ if found | ❌ if not found

**T3-04 — Database/ORM skill** [1 pt]
Look for a skill whose name or description indicates it covers database access, ORM usage, migrations, or query patterns — or matches the detected DB stack.
- ✅ if found | ❌ if not found

**T3-05 — ADRs present** [1 pt]
Look for `docs/adr/` OR `docs/decisions/` containing at least one `.md` file.
- ✅ if found | ❌ if not found

**T3-06 — Domain/pipeline docs** [1 pt]
Look for any `.md` file in `docs/` or `src/` subdirectories that documents domain knowledge, business logic, or a system pipeline — not generic project scaffolding (README, CHANGELOG, CONTRIBUTING, LICENSE, etc.).
- ✅ if at least one domain-specific doc found | ❌ if none

**T3-07 — Subagents defined** [1 pt]
Look for `.claude/agents/` or `.agents/agents/` containing at least one `.md` file with `tools:` or `model:` in its frontmatter. A skills folder does NOT count — subagents have their own tool restrictions or model config, skills do not.
- ✅ if at least one qualifying file found | ❌ otherwise

**T3-08 — Context scoped by path** [1 pt]
Look for any instruction or rules file scoped to a subdirectory (not root) — e.g. an `AGENTS.md`, `.instructions.md`, or `.rules.md` inside a subfolder, or a `.prompt.md` with an `applyTo:` pattern targeting a specific path rather than `"**"`.
- ✅ if found | ❌ if not found

**T3-09 — Finishing/integration workflow skill** [1 pt]
Look for a skill whose name or description indicates it covers finishing a development branch, creating PRs, merging, or integration workflow.
- ✅ if found | ❌ if not found

**T3-10 — MCP server configured** [1 pt]
Look for an MCP server config file (e.g. `.vscode/mcp.json`, `mcp.json`, or similar).
- ✅ if found | ❌ if not found

**T3-11 — Eval/drift config** [1 pt]
Look for any eval dataset or drift monitoring setup — an `evals/` or `eval/` directory with files, or eval config files. This signals the repo can measure whether instructions stay effective as code evolves.
- ✅ if found | ❌ if not found

---

## STEP 4 — SCORE AND GRADE

Tally all points:
- Each ✅ in Tier 1 = 7 pts
- Each ✅ in Tier 2 = 4 pts
- Each ✅ in Tier 3 = 1 pt
- ⚠️ and ❌ = 0 pts
- N/A checks do not affect the denominator

**Normalized score:** `round(raw_pts / max_pts × 100)` — always displayed as `/100` regardless of how many checks are in the audit. This means adding new checks in future never breaks the scale.

Max raw pts: 116 (or 112 if T2-06 is N/A)

| Level | Name | Threshold |
|-------|------|-----------|
| **5** | **Autonomous** | ≥ 90% of possible |
| **4** | **Optimized** | ≥ 75% of possible |
| **3** | **Standardized** | ≥ 55% of possible |
| **2** | **Documented** | ≥ 35% of possible |
| **1** | **Functional** | < 35% of possible |

---

## STEP 5 — GENERATE RECOMMENDATIONS

For every ❌ or ⚠️ check, write a recommendation. Sort by points lost descending (Tier 1 first, then Tier 2, then Tier 3). For each recommendation:

1. State the check ID, name, and points recoverable
2. Give a **direct, agent-executable instruction** — tell the agent exactly what file to create or what content to add, as if the user will say "fix it" and the agent will act immediately. Do NOT include copy-paste blocks for the user — the user's agent will do the work.
3. For partial passes (⚠️), only describe what is **missing** — do not repeat what already exists.
4. For skill gaps, reference the skills registry below

**Tone:**
- For ❌: "Create `<path>` with `<description of content>`"
- For ⚠️: "In `<instruction file>`, add: `<description of missing content only>`"
- For T2-11/12/13/14 ⚠️: list only the specific signals that were NOT found

**Skills registry** (reference when recommending skill creation):

| Category | Suggested skill names |
|----------|-----------------------|
| Dev workflow / guidelines | `dev-guidelines`, `development-process`, `coding-standards` |
| Next.js | `nextjs-patterns`, `next-patterns` |
| React | `react-patterns`, `react-best-practices` |
| Vue | `vue-patterns`, `vue-guidelines` |
| Node/API | `node-patterns`, `api-conventions` |
| Database | `drizzle`, `prisma`, `neon-postgres`, `database-patterns` |
| UI/Design | `ui-ux-pro-max`, `shadcn`, `design-system`, `component-library` |
| Brainstorming | `brainstorming`, `ideation`, `design-thinking` |
| Planning | `writing-plans`, `spec-writing`, `planning` |
| Debugging | `systematic-debugging`, `diagnose`, `debugging` |
| TDD | `tdd`, `test-driven-development` |
| Finishing work | `finishing-a-development-branch`, `pr-workflow`, `git-workflow` |
| Parallelism | `dispatching-parallel-agents`, `subagent-driven-development` |
| Code review | `requesting-code-review`, `receiving-code-review` |

Skill files go in `.agents/skills/<skill-name>/SKILL.md` with frontmatter:
```
---
name: <skill-name>
description: <one sentence — this is what the AI searches to decide when to load it>
---
```

---

## STEP 6 — OUTPUT THE REPORT

### 6a — Print this to chat

```
╔════════════════════════════════════════════════════════════╗
║          AI READINESS AUDIT — <folder name>                ║
║                                                            ║
   ║   Score: XX/100  (XX%)     Level X — Name                  ║
╚════════════════════════════════════════════════════════════╝

DETECTED STACK: <comma-separated list of detected technologies>

🔴 CRITICAL  [XX/49 pts]
   <status> T1-01  Agent instruction file ............. <evidence>
   <status> T1-02  Instruction file has commands ....... <evidence>
   <status> T1-03  Instruction file has conventions .... <evidence>
   <status> T1-04  README exists ........................ <evidence>
   <status> T1-05  README has quick start .............. <evidence>
   <status> T1-06  Environment variable template ........ <evidence>
   <status> T1-07  Env vars documented .................. <evidence>

🔍 FLAGS (non-scoring)
   ℹ️  DIV    Instruction divergence .............. <list files found; "divergent" or "consistent">

🟡 IMPORTANT  [XX/56 pts]
   <status> T2-01  Architecture doc ..................... <evidence>
   <status> T2-02  Tech stack documented ................ <evidence>
   <status> T2-03  Definition of done ................... <evidence>
   <status> T2-04  Single validation command ............. <evidence>
   <status> T2-05  Dev workflow skill ................... <evidence>
   <status> T2-06  Framework skill ...................... <evidence>
   <status> T2-07  Test commands documented .............. <evidence>
   <status> T2-08  Pre-commit hooks configured ........... <evidence>
   <status> T2-09  E2E tests configured .................. <evidence>
   <status> T2-10  TDD workflow documented ............... <evidence>
   <status> T2-11  Operational guardrails ............... <evidence>
   <status> T2-12  Planning protocol .................... <evidence>
   <status> T2-13  Test strategy ........................ <evidence>
   <status> T2-14  Branch finishing protocol ............. <evidence>

🟢 ADVANCED  [XX/11 pts]
   <status> T3-01  UI/Design skill ...................... <evidence>
   <status> T3-02  Brainstorming/planning skill .......... <evidence>
   <status> T3-03  Debugging skill ...................... <evidence>
   <status> T3-04  Database/ORM skill ................... <evidence>
   <status> T3-05  ADRs present ......................... <evidence>
   <status> T3-06  Domain/pipeline docs ................. <evidence>
   <status> T3-07  Subagents defined .................... <evidence>
   <status> T3-08  Context scoped by path ............... <evidence>
   <status> T3-09  Finishing/integration workflow ........ <evidence>
   <status> T3-10  MCP server configured ................ <evidence>
   <status> T3-11  Eval/drift config .................... <evidence>

────────────────────────────────────────────────────────────
TOP RECOMMENDATIONS (sorted by impact)

  1. [+X pts]  <ID> — <title>
               <agent-executable instruction: what file to create/edit and what to add>

  2. [+X pts]  <ID> — <title>
               <agent-executable instruction>

  ... (list all failing checks)

────────────────────────────────────────────────────────────
Report written to: ai-audit-report.md
```

### 6b — Write `ai-audit-report.md` to the repo root

Write a file at `ai-audit-report.md` with this exact structure:

```markdown
# AI Readiness Audit Report

**Project:** <folder name>
**Date:** <today's date>
**Score:** XX/100 (XX%) — Level X: Name

---

## Detected Stack

<comma-separated list>

---

## Results

### 🔴 Critical [XX/49 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T1-01 | Agent instruction file | ✅/❌/⚠️ | <what was found or not found> |
| T1-02 | Instruction file has commands | ... | ... |
| T1-03 | Instruction file has conventions | ... | ... |
| T1-04 | README exists | ... | ... |
| T1-05 | README has quick start | ... | ... |
| T1-06 | Environment variable template | ... | ... |
| T1-07 | Env vars documented | ... | ... |

### 🟡 Important [XX/40 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T2-01 | Architecture doc | ... | ... |
| T2-02 | Tech stack documented | ... | ... |
| T2-03 | Definition of done | ... | ... |
| T2-04 | Single validation command | ... | ... |
| T2-05 | Dev workflow skill | ... | ... |
| T2-06 | Framework skill | ... | ... |
| T2-07 | Test commands documented | ... | ... |
| T2-08 | Pre-commit hooks configured | ... | ... |
| T2-09 | E2E tests configured | ... | ... |
| T2-10 | TDD workflow documented | ... | ... |
| T2-11 | Operational guardrails documented | ... | ... |
| T2-12 | Planning protocol documented | ... | ... |
| T2-13 | Test strategy documented | ... | ... |
| T2-14 | Branch finishing protocol documented | ... | ... |

### 🔍 Flags (non-scoring)

| Flag | Result | Note |
|------|--------|------|
| DIV | ℹ️/N/A | <instruction files found; divergence or consistent> |

### 🟢 Advanced [XX/11 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T3-01 | UI/Design skill | ... | ... |
| T3-02 | Brainstorming/planning skill | ... | ... |
| T3-03 | Debugging skill | ... | ... |
| T3-04 | Database/ORM skill | ... | ... |
| T3-05 | ADRs present | ... | ... |
| T3-06 | Domain/pipeline docs | ... | ... |
| T3-07 | Subagents defined | ... | ... |
| T3-08 | Context scoped by path | ... | ... |
| T3-09 | Finishing/integration workflow | ... | ... |
| T3-10 | MCP server configured | ... | ... |
| T3-11 | Eval/drift config | ... | ... |

---

## Recommendations

### [+X pts] T1-XX — <title>

<agent-executable instruction: what file to create/edit and what content to add. For ⚠️ partial passes, list only the missing signals.>

---

*Generated by [ai-audit](https://github.com/AlonMiz/ai-audit) — install with `npx skills@latest add AlonMiz/ai-audit`*
```

After writing the file, confirm in chat: "Report written to `ai-audit-report.md`."
