# Your AI agent isn't the problem. Your repo is.

You ask your AI coding agent to pick up where you left off. It's a project you touched two weeks ago. You type the task.

It runs `vitest --config vite.config.ts src/` and gets a module resolution error. Tries `npx vitest run`. Gets further, fails on a missing env variable. Your project uses `pnpm validate`. It had nothing to go on. It guessed.

It generates 40 lines of code, marks the task complete, and pushes a commit that fails pre-commit hooks you set up three months ago and apparently never told anyone about. You spend 20 minutes debugging something that wasn't a bug — it was a setup gap.

The agent didn't fail. You handed it an unreadable brief and expected a good result.

**TLDR**
- An AI agent's only memory is your repo's files. If you haven't written something down, it doesn't exist for the agent.
- Most repos fail basic readiness checks — not because developers are careless, but because the checklist is long and no one runs it twice.
- This article walks through 4 checks that cause the most avoidable failures, with real examples from an actual audit.
- There's a command that runs all 35 checks and scores your repo out of 100. It takes about 2 minutes.

---

## Your repo is a brief

An AI agent has no persistent memory. Between sessions, between projects, it knows nothing about your code except what it can read right now.

That means: if you haven't written a rule down, the rule doesn't exist for the agent. Your naming convention. Your test command. The fact that you use `pnpm`, not `npm`. The fact that this project has pre-commit hooks. The fact that "done" means tests pass, not just that the code compiles.

A poorly set-up repo doesn't make the AI stupid. It makes the AI *uninformed*. And uninformed AI invents. It guesses. It uses the default. It marks things complete when they aren't.

---

## What "not ready" actually looks like

**Failure mode 1 — The agent guesses the test command.** No instruction file, or an instruction file with no commands. The agent tries `vitest --config vite.config.ts src/`. Fails. Tries `npx vitest run`. Fails differently. Your project uses `pnpm validate`. Three minutes of debugging a missing script that was never missing.

**Failure mode 2 — The agent commits bad code.** Pre-commit hooks configured, but the agent doesn't know they exist. It commits. The hook rejects it. The agent is confused and tries again with `--no-verify`.

**Failure mode 3 — Done means nothing.** No definition of done in the instruction file. The agent calls the task complete the moment the code runs. No test was run. No linter was run. You find the issue in the next session.

---

## Check 1 — Operational guardrails

This one trips almost every project on the first audit.

The check looks for whether the agent has been explicitly told how to operate safely: no ad-hoc shell commands, no editing files via the terminal, no installing packages without asking, all reusable operations wrapped in named scripts.

A passing project has lines like these in its `AGENTS.md`:

> *"Never inline bash loops in ad-hoc terminal calls"*
> *"create a reusable script in scripts/"*
> *"NEVER git push without explicit permission"*

That's from a real audit. The project scored ✅ because it had at least 3 of the 6 required signals.

Without this, the agent is free to run whatever it decides makes sense in the moment. It might `sed -i` a file instead of using the editor. It might chain 5 piped commands in one terminal call. It might `npm install` a package mid-task. Each is a failure you won't notice until it's already cost you.

The fix is 5 lines in your instruction file. The agent will follow them.

---

## Check 2 — Code quality over time

AI generates volume. It writes fast, writes a lot. It doesn't track what it's already written.

Without automated quality gates, a codebase worked on by AI agents over weeks develops specific pathologies: dead exports no one calls, duplicated logic spread across three files because the agent didn't know it already existed elsewhere, functions with cyclomatic complexity so high no human would have written them by hand.

`fallow` is a static analysis tool that detects dead exports, duplicated logic, and high-complexity functions in one pass. A project that passes this check has it wired into its validate script:

> *`"validate": "npm run type-check && npm run lint && fallow dead-code && fallow dupes && fallow health"`*

That runs on every commit. Unused exports don't survive. Duplicated logic gets flagged before it spreads. Functions that are too complex to safely modify get surfaced before the next agent session adds more on top.

The principle is the same regardless of the tool: the check has to run automatically, not on request.

AI-generated code is not inherently lower quality. But it needs the same guard rails any high-velocity output needs. Without them, you find out about the debt three months later.

---

## Check 3 — TDD

This check is informational — it doesn't affect the score. But it's in the audit because it determines whether you're building something verifiable or just accumulating code that looks tested.

The failure mode is epistemic. If no test failed before the code existed, you have no evidence the test would have caught the bug. The agent writes code, writes a test that passes against that code, calls it done. You have coverage. You have nothing.

A project with TDD properly documented has entries like:

> *"NEVER just change the test to match the current implementation"*

And installed skills like `test-driven-development` and `red-green-refactor` — so the agent knows to reach for them when starting a feature, not verify them after.

---

## Check 4 — Instruction file quality

This one was added recently because it's easy to miss: the instruction file exists, has content, passes all the other checks — and still doesn't work.

Anthropic's own [CLAUDE.md documentation](https://docs.anthropic.com/en/docs/claude-code/memory) is direct: *"target under 200 lines per CLAUDE.md file. Longer files consume more context and reduce adherence."* Their best practices page puts it even more bluntly: *"Bloated CLAUDE.md files cause Claude to ignore your actual instructions."*

The check looks for four signals:
1. **Length ≤ 200 lines** — not a soft guideline, an observed performance threshold
2. **Structured with headers** — a flat bullet list of 80 rules is harder to navigate than 6 headed sections
3. **Concrete commands and paths** — `pnpm validate`, `src/api/`, `lefthook` — not "follow best practices"
4. **No filler** — "write clean code" without a spec attached is noise in the context window

A partial pass on a real project looked like this:

> *T2-23 — ⚠️ Instruction file exists but 3 of 4 signals missing: file is 340 lines, no section headers, contains "write good tests" without a concrete test strategy attached*

That project's agent was technically reading the instruction file. It just wasn't following it — because 340 lines of flat bullets is not a brief anyone follows consistently.

The fix is usually a rewrite, not an addition.

---

Each individual check catches a specific failure mode. Together, they produce a single number.

## The five levels

The audit scores your repo out of 100 and places it on one of five levels:

| Level | Name | Score |
|-------|------|-------|
| 1 | Functional | < 35% |
| 2 | Documented | 35–54% |
| 3 | Standardized | 55–74% |
| 4 | Optimized | 75–89% |
| 5 | Autonomous | ≥ 90% |

Level 4 (the real audit report referenced above scored 84%) means the agent can work with high velocity and low supervision. Level 5 means it can orient itself, plan, execute, verify, and finish without you holding it.

The jump from Level 2 to Level 3 usually takes an afternoon. Level 4 takes a focused session. Level 5 requires deliberate investment — subagents, scoped context, evals.

Not every project needs Level 5. A solo side project you'll touch a dozen times doesn't need the same setup as a team codebase with multiple agents running in parallel. Most projects need to stop being Level 1. Level 3 is enough to stop losing velocity to the same avoidable gaps.

---

## Run the audit

Works with GitHub Copilot, Claude Code, and Cursor — any agent that follows the skills convention.

```
npx skills@latest add AlonMiz/ai-audit
```

Then in your agent:

```
/ai-audit
```

The output looks like this:

```
Score: 84/100 (84%) — Level 4: Optimized

🔴 Foundation [70/70 pts]
✅ T1-01 Agent instruction file — AGENTS.md found at root
✅ T1-09 Definition of done — "Never mark a task as completed without running all three steps"
✅ T1-10 Single validation command — "validate": "npm run type-check && npm run lint && npm run knip"

🟡 Important [40/56 pts]
⚠️  T2-08 Pre-commit hooks — lefthook.yml covers lint and format but no typecheck phase
⚠️  T2-23 Instruction file quality — 340 lines, no section headers
```

35 checks. A score out of 100. A report saved to `reports/` with exactly what's missing and exactly what to add to fix it.

You'll stop losing velocity to the same avoidable gaps on every new project. The checklist is always the same. Now it runs itself.
