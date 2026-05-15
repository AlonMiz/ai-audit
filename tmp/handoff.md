# Handoff — ai-audit skill repo

**Date:** 2026-05-15  
**Repo:** `/home/alon/private/ai-audit` → https://github.com/AlonMiz/ai-audit  
**Next focus:** Improve the audit tool itself (new checks, README polish, example report refresh)

---

## What exists

```
skills/ai-audit/SKILL.md   ← the canonical skill (single source of truth)
README.md                  ← install instructions + check reference tables
examples/ai-audit-report.md ← sample output (from newz, old scoring — needs refresh)
LICENSE                    ← MIT
tmp/                       ← scratch space (this file)
```

Install command: `npx skills@latest add AlonMiz/ai-audit`  
Works with: Claude Code, Cursor, VS Code Copilot (any agent that follows the skills convention).

---

## Current state of SKILL.md

- **28 checks**, **100 pts total** (T1=7×7=49, T2=4×10=40, T3=1×11=11)
- **5-level maturity**: Level 5 Autonomous (≥90%), 4 Optimized (≥75%), 3 Standardized (≥55%), 2 Documented (≥35%), 1 Functional (<35%)
- T2-06 (Framework skill) can be N/A → 96 pts max
- Last commit: `f2ac1e9` — "feat: normalize scoring to 100 pts (T1=7, T2=4, T3=1)"

Scoring confirmed clean — no `[2 pts]`, `[3 pts]`, `/52`, `/21`, `/20` remain.

---

## Known gaps / next tasks

### High priority
1. **README.md check table still shows old weights** — the `## Checks` table has columns like "7 checks × 3 pts = 21 pts". Update to: T1: 7×7=49, T2: 10×4=40, T3: 11×1=11.
2. **examples/ai-audit-report.md is stale** — shows 37/50, Grade C (old scoring). Re-run the audit on `AlonMiz/ai-audit` itself (or another real project) and replace with a fresh 100-pt report.

### Deferred (user said "later")
3. **"Basic rules" checks** — add checks for basic AI safety guardrails in instruction files (e.g. OWASP reminders, "don't delete files without confirmation"). Exact scope TBD.

---

## Suggested skills for next session

```
npx skills@latest add AlonMiz/ai-audit   # the tool itself — to audit this repo
```

---

## Key files to read first

- [skills/ai-audit/SKILL.md](../skills/ai-audit/SKILL.md) — full check definitions, scoring rules, output templates
- [README.md](../README.md) — needs the table weights updated (see gap #1 above)
- [examples/ai-audit-report.md](../examples/ai-audit-report.md) — stale, needs refresh
