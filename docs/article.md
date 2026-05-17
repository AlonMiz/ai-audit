# Your AI agent isn't the problem. Your repo is.

You ask your AI coding agent to pick up where you left off. It's a project you touched two weeks ago. You type the task.

It runs `vitest --config vite.config.ts src/` and gets a module resolution error. Tries `npx vitest run`. Gets further, fails on a missing env variable. Your project uses `pnpm validate`. It had nothing to go on. It guessed.

It generates 40 lines of code, marks the task complete, and pushes a commit that fails pre-commit hooks you set up three months ago and apparently never told anyone about. You spend 20 minutes debugging something that wasn't a bug — it was a setup gap.

The agent didn't fail. You handed it an unreadable brief and expected a good result.

**TLDR**
- An AI agent's only memory is your repo's files. If you haven't written something down, it doesn't exist for the agent.
- Most repos fail basic readiness checks — not because developers are careless, but because the checklist is long and no one runs it twice.
- This article walks through 4 checks that cause the most avoidable failures, with real examples from an actual audit.
- There's a command that runs all 43 checks and scores your repo out of 100. It takes about 2 minutes.

---

## Your repo is a brief

An AI agent has no persistent memory. Between sessions, between projects, it knows nothing about your code except what it can read right now.

That means: if you haven't written a rule down, the rule doesn't exist for the agent. Your naming convention. Your test command. The fact that you use `pnpm`, not `npm`. The fact that this project has pre-commit hooks. The fact that "done" means tests pass, not just that the code compiles.

A poorly set-up repo doesn't make the AI stupid. It makes the AI *uninformed*. And uninformed AI invents. It guesses. It uses the default. It marks things complete when they aren't.

One thing worth saying upfront: the things that make a repo readable for an agent are mostly the same things that make it maintainable for a human. Documented commands. Automated quality gates. Test discipline. Clean, navigable structure. AI readiness isn't a separate category you bolt on — it's repo hygiene with a different name and a concrete checklist.

---

## What "not ready" actually looks like

**Failure mode 1 — The agent commits bad code.** Pre-commit hooks configured, but the agent doesn't know they exist. It commits. The hook rejects it. The agent is confused and tries again with `--no-verify`.

**Failure mode 2 — Done means nothing.** No definition of done in the instruction file. The agent calls the task complete the moment the code runs. No test was run. No linter was run. You find the issue in the next session.

---

## Checks that worth highlighting

43 checks total. Here are four that show up most often in the real reports — with actual audit output.

### Operational guardrails

Does the agent know how to operate safely? No ad-hoc shell commands, no editing files via terminal, no installing packages mid-task. A passing project has explicit rules like:

> *"Never inline bash loops in ad-hoc terminal calls — create a reusable script in scripts/ and add it to package.json/scripts"*
> *"NEVER git push without explicit permission"*

Without this, the agent runs whatever it decides makes sense. `sed -i` a file instead of using the editor. Chain 5 piped commands. Install a package mid-task. Each is a failure you won't notice until it's already cost you.

### Code quality over time

AI writes fast and doesn't track what it's already written. Without automated gates: dead exports, duplicated logic across three files, functions complex enough to be unmaintainable. We recommend [`fallow`](https://github.com/nicolo-ribaudo/fallow) — written in Rust, runs in milliseconds, and covers dead code detection, duplication, and complexity analysis in a single install instead of three separate tools:

> *`"validate": "npm run type-check && npm run lint && fallow dead-code && fallow dupes && fallow health"`*

The check has to run automatically on every commit, not on request. That's the whole point.

### TDD

The old excuse for skipping tests was time. Writing tests was slow. With AI, that excuse is gone. Tests are fast to write now, and the velocity only goes up. What remains is the discipline problem: without red-green-refactor, the agent writes code, then writes a test that passes against that code, and calls it done. You have coverage. You have nothing. The test only proves the code runs — not that it solves the problem you stated. That's what red-green-refactor prevents: it forces a failing test first, so you know what you're actually verifying.

> *"NEVER just change the test to match the current implementation"*

### Instruction file quality

The check most projects miss. The file exists, has content, passes all the other checks — and still doesn't work. Anthropic's own [CLAUDE.md documentation](https://docs.anthropic.com/en/docs/claude-code/memory) is direct: *"target under 200 lines per CLAUDE.md file. Longer files consume more context and reduce adherence."* Four signals: ≤ 200 lines, structured with headers, concrete commands and paths, no filler. A real partial fail:

> *T2-23 — ⚠️ 3 of 4 signals missing: file is 340 lines, no section headers, contains "write good tests" without a concrete test strategy attached*

That agent was technically reading the file. It just wasn't following it. The fix is almost always a rewrite, not an addition.

---

Each check catches a specific failure mode. Together, they produce a single number.

## The five levels

The audit scores your repo out of 100 and places it on one of five levels:

| Level | Name         | Score  |
| ----- | ------------ | ------ |
| 1     | Functional   | < 35%  |
| 2     | Documented   | 35–54% |
| 3     | Standardized | 55–74% |
| 4     | Optimized    | 75–89% |
| 5     | Autonomous   | ≥ 90%  |

Level 4 (the real audit report referenced above scored 84%) means the agent can work with high velocity and low supervision. Level 5 means it can orient itself, plan, execute, verify, and finish without you holding it.

The jump from Level 2 to Level 3 usually takes an afternoon. Level 4 takes a focused session. Level 5 requires deliberate investment — subagents, scoped context, evals.

Not every project needs Level 5. A solo side project you'll touch a dozen times doesn't need the same setup as a team codebase with multiple agents running in parallel. Most projects need to stop being Level 1. Level 3 is enough to stop losing velocity to the same avoidable gaps.

---

## On other tools in this space

There are other tools that do something similar. `npx check-ai`, `aiready-cli` — run them and they scan your repo and tell you what's present. That's a reasonable thing to want.

The difference is what question they're asking. A presence check tells you the file exists. This audit asks whether it works. Is the instruction file structured and concrete, or is it 340 lines of flat bullets the agent technically reads but doesn't follow? Does the validate script cover the right ground? Is the test strategy documented in a way an agent would actually use? Not the same question — not a better one necessarily, just a different one.

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

43 checks. A score out of 100. A report saved to `reports/` with exactly what's missing and exactly what to add to fix it.

You'll stop losing velocity to the same avoidable gaps on every new project. The checklist is always the same. Now it runs itself.
