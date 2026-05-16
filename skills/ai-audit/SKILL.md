---
name: ai-audit
description: "Audit any project's AI readiness. Scores checks across 3 tiers, produces a 5-level maturity rating, and writes ai-audit-report.md with concrete fix recommendations. Run once on a new project or after major changes."
---

You are running an **AI Readiness Audit** on this codebase. Your job is to evaluate how well the project is set up for AI agents to work in it effectively.

**Rules:**
- Read files only. Do NOT modify any project file except writing `ai-audit-report.md` at the end.
- Be systematic: work through every check in order before scoring.
- Be evidence-based: record exactly what you found (filename, line content) for each check. **Never record values from env or secret files** (`.env*`, credential configs) — record only key names or file presence.
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
| `@tanstack/react-query`, `react-query` | React Query | `react-query`, `tanstack`, `query` |
| `@trpc/server`, `@trpc/client` | tRPC | `trpc` |

If `package.json` is absent: mark stack as "unknown" and skip T2-06 and T2-16.

**Keyword matching rule for all skill-based checks:** Match keywords against (1) the skill's YAML frontmatter `description:` field, (2) the `name:` field, or (3) the skill filename/directory name. Full-text content search is NOT required. You must open each skill file to read frontmatter — filename alone is not sufficient.

---

## STEP 3 — RUN ALL 35 CHECKS

Work through every check. For each one record: status (✅ / ❌ / ⚠️), points earned (✅ = full points, ⚠️ = 0, ❌ = 0), and a brief evidence note.

### Status symbols
- ✅ Pass — criteria fully met
- ❌ Fail — criteria not met
- ⚠️ Partial — file exists but content check failed (scores 0; gets a "you have the file, here's what to add" recommendation)

---

### TIER 1 — FOUNDATION (7 pts each, up to 77 pts)

> **Note:** Foundation checks are necessary but not sufficient. Passing all of them means the project has the structural prerequisites for agents to work — not that agents will work well.

**T1-01 — Agent instruction file exists** [7 pts]
Look for any of: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`, `.instructions.md` at the root.
- ✅ if any found | ❌ if none found
- Partial: N/A (binary)

**T1-08 — Instruction file consistency** [7 pts]
Applies only when T1-01 found **2 or more** instruction files. If only 1 (or 0) files were found, mark N/A (excluded from denominator).

When 2+ files exist, read all of them and compare topic by topic across these 5 categories:
1. **Run commands** — test, build, lint, validate, dev commands
2. **Code style / naming rules** — casing, formatting, naming conventions
3. **Tool preferences** — package manager, formatter, linter
4. **Prohibited patterns** — explicit `NEVER` / `do not` rules
5. **Agent behavior rules** — planning, file editing, install gates

- ✅ if 2+ files found AND no contradictions detected across all 5 categories
- ❌ if 2+ files found AND at least one contradiction detected — cite the specific conflicting lines and which files they came from
- N/A if 0 or 1 instruction file found

Note: Duplication (the same rule appearing in both files) is NOT a contradiction and does not score ❌.

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

**T1-09 — Definition of done documented** [7 pts]
Read the instruction file and README. Look for explicit "definition of done" language that includes a reference to a runnable validate or CI command (e.g., `pnpm validate`, `npm run ci`, `make check`).
- ✅ if done definition found AND it references a specific runnable validate/CI command
- ⚠️ if instruction file exists but done definition is absent OR exists without referencing a runnable command
- ❌ if no instruction file

**T1-10 — Single validation command** [7 pts]
Check `package.json` scripts for a key named `validate`, `check`, `ci`, or `test:all` — OR check `Makefile`/`Justfile` for a line matching `^(validate|check|ci|test):`.
- ✅ if found | ❌ if not found
- Partial: N/A (binary)

**T1-11 — Dev workflow skill** [7 pts]
List `.agents/skills/` or `.claude/skills/`. Look for a skill whose name or description indicates it covers development workflow, coding guidelines, or project conventions — the kind of skill an agent should load at the start of every task.

**Matching — two-pass rule:**
1. **Keyword pass:** Check each skill's `name:`, `description:`, and directory name for any of: `workflow`, `guidelines`, `dev`, `process`, `coding-standards`, `conventions`, `superpowers`, `start`.
2. **Judgment pass (only if keyword pass finds nothing):** Re-read each unmatched skill's full `description:` and assess: "Does this skill appear intended to be loaded at the start of every coding task, OR does it describe project-wide conventions an agent must follow?" If yes, treat as a match and note which skill matched and why.

- ✅ if a matching skill found (either pass)
- ⚠️ if skills folder exists but no match found after both passes
- ❌ if no skills folder

---

### TIER 2 — IMPORTANT (4 pts each, up to 64 pts)

**T2-01 — Architecture or context doc** [4 pts]
Look for any documentation that describes how the system is structured — could be a `CONTEXT.md`, `ARCHITECTURE.md`, a `docs/adr/` directory, or any `.md` file in `docs/` whose name or top-level heading indicates it describes the system design, data flow, services, schema, or domain.
- ✅ if any match | ❌ if none found

**T2-02 — Tech stack documented** [4 pts]
Read `README.md` and the instruction file. Look for explicit naming of: the primary framework/language, database, and test tool.
- ✅ if all three categories (framework, DB, testing) are named
- ❌ if README and instruction file are absent or mention fewer than two stack components
- Note: "TypeScript + Next.js + Postgres + Vitest" = ✅; "TypeScript project" alone = ❌

**T2-06 — Framework skill present** [4 pts]
Using the detected **framework** tags from Step 2 (Next.js, React, Vue, Svelte, Angular, Node API only — exclude test tools, ORMs, and databases), check whether a skill exists for EACH detected framework. Match against each skill's frontmatter `name:`, `description:`, and directory name.
- ✅ if every detected framework has a matching skill
- ⚠️ if skills folder exists but one or more framework skills are missing — list which frameworks are unmatched
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

**T2-10 — TDD workflow** [0 pts — informational flag only]
Read the instruction file, README, and skill files' frontmatter. Look for language describing a test-first development process — writing a failing test before writing implementation code, the red-green-refactor loop, or equivalent.
- ℹ️ Note presence or absence. This check does not affect the score — teams that don't use TDD should not be penalized. Record in the report as informational only.

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

**T2-15 — Unused code detection configured** [4 pts]
Check `package.json` devDependencies for at least one unused code / dead-code tool:
- **`fallow`** — covers unused exports, files, dependencies, types, and circular deps; preferred recommendation
- **`knip`** — dead code and unused dependencies
- `depcheck`, `npm-check`, `ts-unused-exports`, `unimported` — narrower unused-dependency tools

Then check `package.json` scripts (or `.fallowrc.json` for fallow) for a script or config that invokes one of those tools.
- ✅ if a qualifying tool is in devDependencies AND a script or config runs it
- ⚠️ if tool is present in devDependencies but no script/config wires it in (or vice-versa)
- ❌ if neither found
- Skip (mark N/A, award 0) if `package.json` is absent

For ❌ recommendations: suggest `fallow` first — `fallow dead-code` covers this check and T2-21 and T2-22 simultaneously.

**T2-21 — Duplication detection configured** [4 pts]
Check `package.json` devDependencies for a duplication detection tool:
- **`fallow`** — `fallow dupes` detects exact, AST-level, and semantic clones; preferred recommendation
- `jscpd` — copy-paste detector

Then check `package.json` scripts (or `.fallowrc.json` for fallow) for a script or config that runs duplication detection.
- ✅ if a qualifying tool is in devDependencies AND a script or config runs it
- ⚠️ if tool is present but no script/config wires it in (or vice-versa)
- ❌ if neither found
- Skip (mark N/A, award 0) if `package.json` is absent

Note: if `fallow` passed T2-15, it also satisfies this check — do not mark ❌ just because there is no separate `jscpd` install.

**T2-22 — Complexity analysis configured** [4 pts]
Check `package.json` devDependencies for a complexity analysis tool:
- **`fallow`** — `fallow health` surfaces high-risk functions, hotspots, and per-file maintainability scores; preferred recommendation
- ESLint rules `complexity` or `sonarjs/cognitive-complexity` configured in `.eslintrc.*` / `eslint.config.*`

Then check `package.json` scripts (or `.fallowrc.json` for fallow) for a script that runs complexity analysis.
- ✅ if a qualifying tool or rule is configured AND wired into a script or CI gate
- ⚠️ if tool/rule is present but not wired in
- ❌ if neither found
- Skip (mark N/A, award 0) if `package.json` is absent

Note: if `fallow` passed T2-15, it also satisfies this check.

**T2-16 — High-impact library skills** [4 pts]
From the detected stack (Step 2), identify any **high-impact libraries** present: React Query, tRPC, Drizzle, Prisma, TypeORM. For each detected library, check whether a skill exists whose frontmatter `name:`, `description:`, or directory name matches the library keyword.
- ✅ if every detected high-impact library has a matching skill (or none are detected)
- ⚠️ if skills folder exists but one or more high-impact library skills are missing — list which libraries are unmatched
- ❌ if no skills folder and at least one high-impact library is detected
- Skip (mark N/A, award 0) if `package.json` is absent

**T2-17 — Language discipline documented** [4 pts]
Detect the primary language from `package.json` (presence of `typescript` in devDependencies, or a `tsconfig.json` at root = TypeScript). Read the instruction file and check for language-specific guardrails:
- **TypeScript projects** — look for ALL THREE: (1) prohibition of `any` (e.g. "no any", "avoid any", "ban any"), (2) prohibition of type assertions (e.g. "no `as`", "avoid type assertions", "no casting"), (3) reference to `strict` mode or `strictNullChecks`
- **JavaScript-only projects** — look for at least one explicit convention (e.g. "no var", "use ESM", "always use strict")

Scoring:
- ✅ all required signals found for the detected language
- ⚠️ if instruction file exists but one or more signals are missing — list the missing ones
- ❌ if no instruction file

**T2-18 — Schema validation configured** [4 pts]
Check `package.json` dependencies (not devDependencies) for a runtime schema validation library: `zod`, `valibot`, `arktype`, `@sinclair/typebox`, `yup`, `joi`, or `superstruct`. Then read the instruction file and check for language directing agents to use schema validation at system boundaries (e.g. "validate all external inputs with Zod", "use schema validation for API responses", "parse env vars with zod").
- ✅ if a qualifying library is in dependencies AND the instruction file directs agents to use it
- ⚠️ if library is present but instruction file has no guidance (or guidance exists but no library) — describe what is missing
- ❌ if neither found

**T2-19 — No duplicate content across instruction files** [4 pts]
Applies only when T1-01 found **2 or more** instruction files. If only 1 (or 0) files were found, mark N/A (excluded from denominator).

Using the same 5 categories as T1-08, scan for substantive blocks repeated across files:
1. **Run commands** — same command listed in both files
2. **Code style / naming rules** — same rule block repeated verbatim or near-verbatim
3. **Tool preferences** — same tool choice stated in both files
4. **Prohibited patterns** — same `NEVER` / `do not` rule duplicated
5. **Agent behavior rules** — same planning/editing/install rule repeated

A block is "substantive" if it is 3 or more lines, or a named rule/command. A single short sentence (e.g. "use TypeScript") repeated in both files does NOT count as duplication.

- ✅ if 2+ files found AND no substantive duplication detected
- ⚠️ if 2+ files found AND 1–2 categories have substantive duplication — list which categories and files
- ❌ if 2+ files found AND 3+ categories are duplicated — list all affected categories and files
- N/A if 0 or 1 instruction file found

Note: This check is about wasted context budget, not correctness. Duplication here does NOT imply contradiction (T1-08 handles that).

**T2-20 — Code collaboration protocol documented** [4 pts]
Read the instruction file. Look for at least 2 of these 3 concerns addressed:
1. **Commit conventions** — commit message format, scope, or style (e.g. Conventional Commits, atomic commits, `feat:`/`fix:` prefixes, what makes a good commit)
2. **PR/MR protocol** — what a pull/merge request should contain (description, checklist, title format, draft vs ready, linked issue)
3. **Code review process** — how to give or receive review feedback, what must be addressed before merge, or a code review skill referenced

Scoring:
- ✅ 2 or more concerns addressed
- ⚠️ 0–1 concerns addressed and instruction file exists — recommendation lists the missing concerns
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
Look for any of these patterns indicating a task-scoped agent with restricted capabilities:
- **Claude/Copilot agents:** `.claude/agents/` or `.agents/agents/` — any `.md` file with `tools:` or `model:` in its frontmatter
- **Prompt-mode agents:** any `.prompt.md` file in the repo with `mode: agent` in its YAML frontmatter
- **Cursor scoped rules:** any `.mdc` file with `alwaysApply: false` in its frontmatter (indicating a task-scoped rule, not a global one)

A skills folder (`.agents/skills/`) does NOT count — subagents have their own tool restrictions or model config, skills do not.
- ✅ if at least one qualifying pattern found | ❌ otherwise

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

**T3-12 — Architecture boundaries configured** [1 pt]
Check for explicit architecture boundary enforcement — import rules between layers or modules that prevent drift over time:
- **`fallow`** with a `boundaries` config in `.fallowrc.json` (preset: `bulletproof`, `layered`, `hexagonal`, `feature-sliced`, or custom zones/rules)
- **`eslint-plugin-boundaries`** or **`@typescript-eslint/no-restricted-imports`** rules configured in ESLint config
- Any other tool or config enforcing module-level import restrictions

- ✅ if boundary enforcement is configured | ❌ if not found
- Skip (mark N/A, award 0) if `package.json` is absent

---

## STEP 4 — SCORE AND GRADE

Tally all points:
- Each ✅ in Tier 1 = 7 pts
- Each ✅ in Tier 2 = 4 pts
- Each ✅ in Tier 3 = 1 pt
- ⚠️ and ❌ = 0 pts
- N/A checks do not affect the denominator

**Normalized score:** `round(raw_pts / max_pts × 100)` — always displayed as `/100` regardless of how many checks are in the audit. This means adding new checks in future never breaks the scale.

Max raw pts: 163 when T1-08 and T2-19 both apply (2+ instruction files found), 149 when both are N/A (≤1 file). Subtract 4 more for each of T2-06/T2-15/T2-16/T2-21/T2-22 that is also N/A (no package.json). Subtract 1 more if T3-12 is N/A.

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
4. For skill gaps, **first check if an existing published skill covers the need** — run `npx skills@latest search <keyword>` or browse https://www.skills.sh. If a matching skill exists, recommend installing it (`npx skills@latest add <author>/<repo>`) instead of creating one from scratch. Only recommend creating a new skill when no published skill matches.

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
| React Query | `react-query-patterns`, `tanstack-query` |
| tRPC | `trpc-patterns`, `trpc-conventions` |
| Schema validation | `schema-validation`, `zod-patterns` |

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

🔴 FOUNDATION  [XX/XX pts]
   <status> T1-01  Agent instruction file ............. <evidence>
   <status> T1-02  Instruction file has commands ....... <evidence>
   <status> T1-03  Instruction file has conventions .... <evidence>
   <status> T1-04  README exists ........................ <evidence>
   <status> T1-05  README has quick start .............. <evidence>
   <status> T1-06  Environment variable template ........ <evidence>
   <status> T1-07  Env vars documented .................. <evidence>
   <status> T1-08  Instruction file consistency ......... <N/A if ≤1 file; else list files and evidence>
   <status> T1-09  Definition of done ................... <evidence>
   <status> T1-10  Single validation command ............. <evidence>
   <status> T1-11  Dev workflow skill ................... <evidence>

🟡 IMPORTANT  [XX/72 pts]
   <status> T2-01  Architecture doc ..................... <evidence>
   <status> T2-02  Tech stack documented ................ <evidence>
   <status> T2-06  Framework skill ...................... <evidence>
   <status> T2-07  Test commands documented .............. <evidence>
   <status> T2-08  Pre-commit hooks configured ........... <evidence>
   <status> T2-09  E2E tests configured .................. <evidence>
   ℹ️ T2-10  TDD workflow (informational) .............. <present or absent — does not affect score>
   <status> T2-11  Operational guardrails ............... <evidence>
   <status> T2-12  Planning protocol .................... <evidence>
   <status> T2-13  Test strategy ........................ <evidence>
   <status> T2-14  Branch finishing protocol ............. <evidence>
   <status> T2-15  Unused code detection ................ <evidence>
   <status> T2-16  High-impact library skills ............ <evidence>
   <status> T2-17  Language discipline ................... <evidence>
   <status> T2-18  Schema validation .................... <evidence>
   <status> T2-19  No duplicate instruction content ...... <N/A if ≤1 file; else list any duplicated categories>
   <status> T2-20  Code collaboration protocol .......... <evidence>
   <status> T2-21  Duplication detection ................ <evidence>
   <status> T2-22  Complexity analysis .................. <evidence>

🟢 ADVANCED  [XX/12 pts]
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
   <status> T3-12  Architecture boundaries .............. <evidence>

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
**Date:** <today's date, YYYY-MM-DD format>
**Audit Version:** 0.0.3
**Score:** XX/100 (XX%) — Level X: Name

---

## Detected Stack

<comma-separated list>

---

## Results

### 🔴 Foundation [XX/XX pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T1-01 | Agent instruction file | ✅/❌/⚠️ | <what was found or not found> |
| T1-02 | Instruction file has commands | ... | ... |
| T1-03 | Instruction file has conventions | ... | ... |
| T1-04 | README exists | ... | ... |
| T1-05 | README has quick start | ... | ... |
| T1-06 | Environment variable template | ... | ... |
| T1-07 | Env vars documented | ... | ... |
| T1-08 | Instruction file consistency | ✅/❌/N/A | <N/A if ≤1 file; else list files and any contradictions found> |
| T1-09 | Definition of done | ... | ... |
| T1-10 | Single validation command | ... | ... |
| T1-11 | Dev workflow skill | ... | ... |

### 🟡 Important [XX/72 pts]

| ID | Check | Status | Evidence |
|----|-------|--------|----------|
| T2-01 | Architecture doc | ... | ... |
| T2-02 | Tech stack documented | ... | ... |
| T2-06 | Framework skill | ... | ... |
| T2-07 | Test commands documented | ... | ... |
| T2-08 | Pre-commit hooks configured | ... | ... |
| T2-09 | E2E tests configured | ... | ... |
| T2-10 | TDD workflow (informational) | ℹ️ | <present or absent — does not affect score> |
| T2-11 | Operational guardrails documented | ... | ... |
| T2-12 | Planning protocol documented | ... | ... |
| T2-13 | Test strategy documented | ... | ... |
| T2-14 | Branch finishing protocol documented | ... | ... |
| T2-15 | Unused code detection configured | ... | ... |
| T2-16 | High-impact library skills | ... | ... |
| T2-17 | Language discipline documented | ... | ... |
| T2-18 | Schema validation configured | ... | ... |
| T2-19 | No duplicate instruction content | ✅/⚠️/❌/N/A | <N/A if ≤1 file; else list duplicated categories and files> |
| T2-20 | Code collaboration protocol documented | ... | ... |
| T2-21 | Duplication detection configured | ... | ... |
| T2-22 | Complexity analysis configured | ... | ... |

### 🟢 Advanced [XX/12 pts]

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
| T3-12 | Architecture boundaries configured | ... | ... |

---

## Recommendations

### [+X pts] T1-XX — <title>

<agent-executable instruction: what file to create/edit and what content to add. For ⚠️ partial passes, list only the missing signals.>

---

*Generated by [ai-audit](https://github.com/AlonMiz/ai-audit) v0.0.3 — install with `npx skills@latest add AlonMiz/ai-audit`*
```

After writing the file, confirm in chat: "Report written to `ai-audit-report.md`."

---

## STEP 7 — OFFER TO FIX

Immediately after confirming the report, ask the user what to do next using the ask-questions tool. Present exactly these four options:

- **Fix Critical + Important gaps** — applies all mechanical fixes for Tier 1 (⚠️ checks) and Tier 2 (❌ and ⚠️ checks): updating the instruction file with missing sections, creating skill stubs, fixing hook configs. Does not touch Tier 3.
- **Fix all mechanical gaps** — same as above, plus Tier 3 fixes that require no domain judgment: create skill stubs, add `applyTo` frontmatter to existing prompts, scaffold a subagent file, add typecheck phase to pre-commit hooks, create an MCP config stub.
- **Walk me through each fix** — present fixes one at a time in order of points lost, waiting for confirmation before each.
- **Nothing, the report is enough** — stop here.

**Mechanical vs judgment distinction — apply this before acting:**

The following checks cannot be auto-applied without user input. For these, describe what is needed and ask before writing anything:
- **T3-05 (ADRs)** — requires the user to supply the actual architectural decisions to record.
- **T3-11 (Eval/drift config)** — requires the user to define the evaluation strategy.
- Any check where the correct content cannot be inferred from existing repo files.

**If the user chooses option 1 or 2:**
1. Group related fixes into chunks of up to 5 changes (e.g. all AGENTS.md additions in one chunk, all new skill files in one chunk).
2. Apply each chunk sequentially — do not parallelize writes across unrelated files.
3. After all fixes are applied, re-run STEP 1–6 to confirm the score improved and report the new score.
4. For any judgment-required check that was skipped, explicitly tell the user what they need to provide to earn those points.

**If the user chooses option 3:**
Present each failing check one at a time: ID, title, points available, and the specific fix. Wait for the user to say "yes", "skip", or provide context before moving to the next fix.
