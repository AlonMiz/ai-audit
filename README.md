# ai-audit

An agent skill that audits any codebase's AI readiness in one command.

Scores what matters: the context an agent needs to orient itself, the workflow knowledge that drives quality and velocity, and the domain depth that prevents recurring mistakes.

Outputs a **5-level maturity rating** (score normalized to /100) and writes `ai-audit-report.md` with concrete fix recommendations.

## Install

```bash
npx skills@latest add AlonMiz/ai-audit
```

Then in your agent (Claude Code, Cursor, VS Code Copilot, etc.):

```
/ai-audit
```

The agent reads your project files, runs all checks, prints a scored report to chat, and saves `ai-audit-report.md` at your repo root.

## What it checks

### 🔴 Foundation — up to 11 checks × 7 pts = up to 77 pts

The basics. Without these, an AI agent has no reliable orientation — and no consistent way to know when it's done or how to work.

| ID | Check |
|----|-------|
| T1-01 | Agent instruction file (`AGENTS.md`, `CLAUDE.md`, `.cursorrules`, etc.) |
| T1-02 | Instruction file has run commands |
| T1-03 | Instruction file has coding conventions |
| T1-04 | README exists |
| T1-05 | README has quick start |
| T1-06 | Environment variable template (`.env.example`) |
| T1-07 | Env vars are documented with comments |
| T1-08 | Instruction files are consistent — no contradictions *(conditional: applies when 2+ instruction files exist)* |
| T1-09 | Definition of done documented |
| T1-10 | Single validation command (`validate`, `ci`, `check`) |
| T1-11 | Dev workflow skill |

### 🟡 Important — up to 19 checks × 4 pts = up to 76 pts

Without these, the AI will make recurring mistakes on conventions and workflow.

| ID | Check |
|----|-------|
| T2-01 | Architecture or context doc |
| T2-02 | Tech stack documented |
| T2-06 | Framework skill (auto-detected from `package.json`) *(conditional: N/A if no package.json)* |
| T2-07 | Test commands documented (unit + E2E) |
| T2-08 | Pre-commit hooks configured (lint + format + typecheck) |
| T2-09 | E2E tests configured |
| T2-10 | TDD workflow *(informational only — does not affect score)* |
| T2-11 | Operational guardrails documented |
| T2-12 | Planning protocol documented |
| T2-13 | Test strategy documented |
| T2-14 | Branch finishing protocol documented |
| T2-15 | Unused code detection configured (fallow, knip, etc.) |
| T2-16 | High-impact library skills (React Query, tRPC, Drizzle, etc.) *(conditional: N/A if no package.json)* |
| T2-17 | Language discipline documented (TypeScript: no `any`, no `as`, strict mode) |
| T2-18 | Schema validation configured (Zod, Valibot, etc.) |
| T2-19 | No duplicate content across instruction files *(conditional: applies when 2+ instruction files exist)* |
| T2-20 | Code collaboration protocol documented (commits, PRs, code review) |
| T2-21 | Duplication detection configured (fallow, jscpd) |
| T2-22 | Complexity analysis configured (fallow health, ESLint complexity) |
| T2-23 | Instruction file quality (≤200 lines, headers, concrete commands, no filler) |

### 🟢 Advanced — 12 checks × 1 pt = 12 pts

Domain knowledge loaded on demand via skills.

| ID | Check |
|----|-------|
| T3-01 | UI/Design skill |
| T3-02 | Brainstorming/planning skill |
| T3-03 | Debugging skill |
| T3-04 | Database/ORM skill |
| T3-05 | ADRs present |
| T3-06 | Domain/pipeline docs |
| T3-07 | Subagents defined |
| T3-08 | Context scoped by path |
| T3-09 | Finishing/integration workflow skill |
| T3-10 | MCP server configured |
| T3-11 | Eval/drift config |
| T3-12 | Architecture boundaries configured (fallow boundaries, eslint-plugin-boundaries) *(conditional: N/A if no package.json)* |

## Maturity levels

| Level | Name | Threshold |
|-------|------|-----------|
| 5 | Autonomous | ≥ 90% |
| 4 | Optimized | ≥ 75% |
| 3 | Standardized | ≥ 55% |
| 2 | Documented | ≥ 35% |
| 1 | Functional | < 35% |

## Example output

See [examples/ai-audit-report.md](examples/ai-audit-report.md) for a real report generated on a Next.js project.

## How it works

The skill is a single `SKILL.md` file. When invoked, your agent:

1. Reads key project files (instruction files, README, package.json, skill folders, docs, config files) — never touches `src/`
2. Detects the stack from `package.json`
3. Runs all 43 checks in order, recording evidence for each
4. Scores and grades the result
5. Generates ranked recommendations with copy-pasteable fixes
6. Writes `ai-audit-report.md` and prints a summary to chat

## Background

[Your AI agent isn't the problem. Your repo is.](docs/article.md) — the story behind this tool, why AI-unready repos cause avoidable failures, and a walkthrough of 4 checks that matter most.

## License

MIT
