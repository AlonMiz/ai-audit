# ai-audit

An agent skill that audits any codebase's AI readiness in one command.

Scores **28 checks** across 3 tiers (100 pts max), outputs a **5-level maturity rating**, and writes `ai-audit-report.md` with concrete, copy-pasteable fix recommendations.

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

### 🔴 Critical — 7 checks × 7 pts = 49 pts

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

### 🟡 Important — 10 checks × 4 pts = 40 pts

Without these, the AI will make recurring mistakes on conventions and workflow.

| ID | Check |
|----|-------|
| T2-01 | Architecture or context doc |
| T2-02 | Tech stack documented |
| T2-03 | Definition of done documented |
| T2-04 | Single validation command (`validate`, `ci`, `check`) |
| T2-05 | Dev workflow skill |
| T2-06 | Framework skill (auto-detected from `package.json`) |
| T2-07 | Test commands documented (unit + E2E) |
| T2-08 | Pre-commit hooks configured (lint + format + typecheck) |
| T2-09 | E2E tests configured |
| T2-10 | TDD workflow documented |

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

### 🔍 Non-scoring flags

| Flag | What it checks |
|------|---------------|
| DIV | Instruction file divergence — detects contradictions when multiple instruction files exist |

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
3. Runs all 28 checks in order, recording evidence for each
4. Scores and grades the result
5. Generates ranked recommendations with copy-pasteable fixes
6. Writes `ai-audit-report.md` and prints a summary to chat

## License

MIT
