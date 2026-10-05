---
name: claudex-route
description: "Recommend a model and a scoped handoff for a task in Claude Code or Codex, and obtain second opinions or run a discussion round on a decision. Use when choosing who should handle a task, weighing a decision, seeking a second opinion, getting unstuck, or delegating focused work; execute one handoff or one discussion round when requested."
---

# Claudex Route

Pick the next step: stay, second opinion, discussion round, investigate blocker, or delegate a bounded piece. Recommend briefly before executing. Rules only here; figures, history, CLI details in `references/` (read on demand).

## Job

- Identify host, current model, outcome, constraints. Weigh ambiguity, dependencies, verification difficulty, context size, cost/speed.
- Keep explicit model/provider choices of the user.
- Advice request = recommendation only. "Route and do it" = scoped handoff within existing permissions. Skill invocation alone authorizes nothing.
- Ask only if a missing fact changes the recommendation; else state the assumption.

## Role first

| Situation | Step |
|---|---|
| Simple, enough context | Stay - but Opus host never builds |
| Consequential/ambiguous plan ready | Second opinion before building |
| Implementation ready | Second opinion on changes vs requirements, incl. test validity |
| Repeated failures | Other provider gets repro, evidence, failed attempts; asks for testable alternative |
| Separable task, clear inputs + acceptance | Delegate to smaller model, inspect result |
| Real decision, wrong choice expensive to undo | Discussion round Opus vs Astra; agreement or dissent to user |

- Delegate by default. Opus host: writes brief, delegates, accepts. Does not build.
- Limit = specification cost: do not split work smaller than its brief. Limits splitting, not the no-build rule.
- Reviewer != author: minimum fresh session, better other model, best other provider. Closer reviewer -> criteria fixed in advance.

## Models

Cost per task: Luna < Sol < Astra; Sonnet < Opus < Fable. Luna/Sol/Astra = Codex CLI, OpenAI allowance. Sonnet/Opus/Fable = Anthropic allowance (spare it). Flash = MiniMax subscription, billed per request (per user), Claude CLI profile `minimax-m3`. Unlisted models (incl. `MiniMax-M3` without Flash): not used. Haiku: never; built-in agents defaulting to it (e.g. Explore) get explicit `sonnet`, or the sweep goes to Sol/Flash/Luna.

| Model | Effort | For | Not for |
|---|---|---|---|
| **Sol** `gpt-6.1-sol` | `low` | Default builder: clear changes/implementation as ONE undivided brief. All bulk and diligence work, whoever coordinates: reading, sweeps, inventories, summaries, routine backend, refactoring, docs, reports, documents, plans, analyses. No allowance threshold. | Architecture decisions. Unknown-cause defects. Never `high`. `medium` only if caller measured it for that job. |
| **Flash** `MiniMax-M3.1-Flash-Preview` | `low` | Builder beside Sol for clear build and diligence: implementation, known-cause fixes, refactoring, docs, inventories, summaries, reports. Sol vs Flash: freer allowance decides; Codex empty -> Flash. Per-request billing: one large coherent brief (all paths, steps, acceptance) that runs in few turns; never piecemeal. Host re-checks quotes, figures, facts. Concepts only as draft, Opus checks. | Judgement, gates, acceptance, assignment/classification quality, unknown-cause search, security code. |
| **Luna** `gpt-6-luna` | `high` | Narrow, repeatable, machine-checkable: fixtures, extraction, small helpers, machine-decidable checks. | Judgement checks. Anything without stated acceptance criterion. Never `low`/`medium` (stops before editing). |
| **Astra** `gpt-6-astra` | `low` build/review, `high` debate | Hard builds: unknown cause, deep debugging taken whole, demanding logic, tracing a value through code. End review of risky builds - once, at the end, read-only. Risky = close to money, gate/security code, field/interface renames with silent consumers. Other side in second opinions and discussion rounds. | Clear builds. End review on every build. Coordinating sub-workers (double cost, no time gain). |
| **Opus** `claude-opus-5-5` | `high` | Delegate, accept, judge. One side of every discussion round. | Any building. Bulk reading. Condensing large input per task. |
| **Sonnet** `claude-sonnet-5-5` / `sonnet` | `low` code, `high` documents | Clear assignments where a Claude model is technically required: agent frontmatter, Claude subagents, workflow agents. | Cause-finding, value tracing, review/acceptance judgement. Very large reading loads. Bulk. |
| **Fable** `claude-fable-5-1` | - | Conception, decomposition, schemas, security concepts, roadmaps. From Codex: second opinion on conception. | Execution after design is agreed. |

Order per work slot:

1. Claude model technically required? -> `sonnet`; if cause-finding, judgement or close to money -> `opus`.
2. Else Codex or Flash: clear build, diligence -> Sol or Flash `low` (freer allowance); documents, plans, analyses -> Sol `low`; small + machine-checkable -> Luna `high`; genuinely hard -> Astra `low`.
3. Hard only because vague -> clarify first (goal, inputs as paths, steps, output format, machine-checkable acceptance), then Sol.
4. Opus at no build slot. Only delegating, accepting, judging - incl. Claude-only slots of step 1 with cause-finding, judgement or money.

Balance: keep 7-day usage of both subscriptions equal within the scopes. Gap <= 5 points: scopes decide. Larger: heavier side gives way where scopes overlap. Sol and Flash exempt, no threshold. Never build work back to Opus. Figure unknown -> route by scope, say so. Where to read figures, freshness, availability, prices: [kontingent.md](references/kontingent.md).

Evidence (bench checks, credits, price comparison, CLI versions): [messbelege.md](references/messbelege.md). Read only when a choice is contested or a figure is asked. Never quote figures from memory.

## Second opinion / discussion round

Choice with real options + expensive to undo -> do not settle alone. Both forms read-only, between providers.

- **Second opinion** (1 round): fresh session of other provider's strongest model (from Claude Code: Astra `high`; from Codex: Opus `high`, Fable for conception). Gets question, options, evidence as paths, criteria fixed in advance. For: plan before build, implementation before acceptance, diagnosis before expensive fix. Host states own position first, then arbitrates each objection against evidence. Perspective, not approval.
- **Discussion round** (max 3 rounds, `debatte-v1`): Opus `high` vs Astra `high`, equal sides. Agreement = same option, no open objection of severity `hoch`. Dissent -> user, both final positions. Never manufacture convergence.

Before either: read [debatte-v1.md](references/debatte-v1.md) (setup, schema [debatte-zug-v1.json](references/debatte-zug-v1.json), briefs, CLI calls, failure rules, record). Pass it to subagents by path. Moderator: no vote, no paraphrasing.

Keep forms apart (rounds are expensive): measurable -> measure. Small + reversible -> host decides, states assumption. One plan/diff -> second opinion. Real decision -> discussion round. User's rules and rights: never debated. Author never gives second opinion on own work.

## Routing brief (< 200 words)

- **Recommendation:** stay / named provider+model for a role / second opinion / discussion round.
- **Why:** 1-2 task-specific reasons, plus material uncertainty.
- **Handoff:** bounded assignment, context/files, expected result, permitted actions, success check. If staying: next step.

Max one alternative. No interview, no model catalog. Never claim the session's model changed; a child CLI call is a separate session.

## Execute one handoff

Read [handoff.md](references/handoff.md) first. Core:

- Provider CLI from project dir; model and effort explicit (Codex: `-m`, `-c model_reasoning_effort="..."`).
- Self-contained brief via stdin or file; child inherits nothing.
- Read-only enforced by sandbox, not by prompt wording. Never bypass permissions.
- Fresh session, bounded timeout, separate output files outside the project.
- Empty output, timeout, permission failure = not success: report, no auto-retry, no escalation.
- Inspect result, run checks, report model requested vs observed. Stop after this one handoff.
