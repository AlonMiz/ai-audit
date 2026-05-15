# ai-audit

An agent skill that audits any codebase's AI readiness in one command.

Scores **34 checks** across 3 tiers, outputs a **5-level maturity rating** (score normalized to /100), and writes `ai-audit-report.md` with concrete fix recommendations.

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

### 🔴 Foundation — up to 8 checks × 7 pts = up to 56 pts

The basics. Without these, an AI agent has no reliable orientation in your project.

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

### 🟡 Important — up to 18 checks × 4 pts = up to 72 pts

Without these, the AI will make recurring mistakes on conventions and workflow.

| ID | Check |
|----|-------|
| T2-01 | Architecture or context doc |
| T2-02 | Tech stack documented |
| T2-03 | Definition of done documented |
| T2-04 | Single validation command (`validate`, `ci`, `check`) |
| T2-05 | Dev workflow skill |
| T2-06 | Framework skill (auto-detected from `package.json`) *(conditional: N/A if no package.json)* |
| T2-07 | Test commands documented (unit + E2E) |
| T2-08 | Pre-commit hooks configured (lint + format + typecheck) |
| T2-09 | E2E tests configured |
| T2-10 | TDD workflow *(informational only — does not affect score)* |
| T2-11 | Operational guardrails documented |
| T2-12 | Planning protocol documented |
| T2-13 | Test strategy documented |
| T2-14 | Branch finishing protocol documented |
| T2-15 | Code quality tools configured (knip, depcheck, etc.) |
| T2-16 | High-impact library skills (React Query, tRPC, Drizzle, etc.) *(conditional: N/A if no package.json)* |
| T2-17 | Language discipline documented (TypeScript: no `any`, no `as`, strict mode) |
| T2-18 | Schema validation configured (Zod, Valibot, etc.) |
| T2-19 | No duplicate content across instruction files *(conditional: applies when 2+ instruction files exist)* |

### 🟢 Advanced — 11 checks × 1 pt = 11 pts

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
3. Runs all 34 checks in order, recording evidence for each
4. Scores and grades the result
5. Generates ranked recommendations with copy-pasteable fixes
6. Writes `ai-audit-report.md` and prints a summary to chat

## License

MIT
