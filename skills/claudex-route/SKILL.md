---
name: claudex-route
description: "Recommend a model and a scoped handoff for a task in Claude Code or Codex. Use when choosing who should handle a task, seeking a second opinion, getting unstuck, or delegating focused work; execute one handoff when requested."
---

# Claudex Route

Choose a useful next step for the task: keep it with the current agent, obtain a second opinion, investigate a blocker, or delegate a bounded piece of work. Give a brief recommendation before any requested execution. This skill is self-contained and does not start Claudex Loop, require a formal plan, or create review logs.

## Understand the job

Use the current conversation and just enough relevant project context to identify the host, current model when known, desired outcome, and constraints. Consider ambiguity, code dependencies, verification difficulty, context size, and cost or speed preferences. A short prompt can describe a difficult task. Preserve explicit model and provider choices; do not silently replace them with your preferred pairing.

Treat a request for advice as recommendation-only. A request to route and perform work authorizes the scoped handoff within the user's existing permissions. The skill invocation by itself does not authorize implementation. Ask a question only when a missing fact materially changes the recommendation or authorized work; otherwise state your assumption.

## Choose the role before the model

| Situation | Useful next step |
|---|---|
| Straightforward task, sufficient context, no reason to delegate | Stay with the current agent; avoid handoff overhead |
| A consequential or ambiguous plan is ready | Ask another provider to challenge requirements, assumptions, and acceptance criteria before building |
| An implementation is ready | Ask another provider to inspect relevant changes against requirements, including test validity |
| Repeated attempts have failed | Give another provider the reproduction, evidence, and failed approaches; ask for a testable alternative explanation |
| A separable task has clear inputs and acceptance checks | Delegate that piece to a suitable smaller model and inspect its result |
| The user wants repeated planning, revision, building, and independent inspection | Recommend Claudex Loop; load that separate skill only if the user requests that workflow |

The current agent keeps the user's requirements and coordinates the work. A different provider offers another perspective, not guaranteed correctness. A cheaper delegate does not need to be a peer reviewer of the entire architecture.

Delegate by default and treat doing the work yourself as the exception that needs a reason: whoever takes a task hands out bounded sub-assignments with clear inputs and acceptance checks. The limit is specification cost. Writing a precise assignment is itself work, so delegation pays only when the task is substantially larger than the brief that describes it; splitting small work costs more than it saves.

Prefer a reviewer who did not write the work. Independence is a gradient, not a switch - same session, then a fresh session of the same model, then a different model, then a different provider - so move as far along it as is practical, and at minimum use a fresh session. The reason is plausible but unproven: a model tends to re-read its own work with the assumptions it wrote it with. Treat it as a precaution, not an established fact, and the closer reviewer and author stand, the more the review must hang on external criteria fixed in advance.

## Select a practical candidate

Start with models available in the user's environment. The list below records the user's own routing preferences (September 2026), not a permanent leaderboard. By cost per task: Luna < Terra < Astra < Opus < Fable. Luna, Terra and Astra run through the Codex CLI on a subscription allowance; Opus and Fable bill real money. Each entry gives a short scope and then what the model should not be used for - exclusions age better than task lists and also decide cases nobody listed.

- **GPT-5.6 Luna** (`gpt-5.6-luna`), cheapest: narrow, repeatable work with a machine-checkable result - fixtures, extraction, doc updates, small isolated helpers - and checks a machine can decide: a test run, schema validity, completeness against a given list, diff size. *Never* for judgement checks such as "is this code good" or "is the architecture sound": a "looks fine" from a model too weak to judge reads like an approval and is worse than no review at all. *Not* when no acceptance criterion can be stated.
- **GPT-5.6 Terra** (`gpt-5.6-terra`): standard coding that needs more judgement and context than a Luna task - routine backend work, refactoring, comments and documentation, index building and file summaries. *Not* for architecture decisions, and *not* for defects whose cause is still unknown.
- **GPT-6 Astra** (`gpt-6-astra`): hard but clearly stated assignments - complex algorithms, performance work, demanding logic - and deep debugging, which goes to it whole instead of being split: the effort sits in finding the cause, the patch is often one line, and before the diagnosis nobody knows what the parts would be. From Claude Code it is the candidate for cross-provider review. *Not* for ordinary mid-sized work that Terra handles - the allowance spent there buys nothing and is missing from the work that needs it. *Not* when the assignment itself is still unclear, or when the input exceeds the context window. Where Astra wrote the code, Astra is not its reviewer: the more it executes, the less it can check, and the review goes back to another model.
- **Claude Opus** (`claude-opus-5`): work whose difficulty lies in *volume* rather than logic - legacy code bases, long specifications, assessments spanning many files. The distinguishing feature against Astra is the context window, not model strength; with a small input Astra is the cheaper route. *Not* on repository size alone, which does not prove a need for that much context at once. *Not* for condensing a large input per task so it fits a smaller window - reading the full context is exactly the expensive part, so the saving is cancelled; only an index built once and reused is worth it, and building it belongs to Terra or Luna.
- **Claude Fable 5.1** (`claude-fable-5-1`), most expensive: conception and decomposition - cutting large undertakings into precise assignments, designing schemas, security concepts and roadmaps. From Codex it is the candidate for cross-provider review. *Not* for execution once a design is agreed: from there on the work is derivation, and a cheaper model does it from the stated brief.
- **Sonnet and Haiku are not used** in this environment - not as builder, reviewer, inspector or research subagent. Built-in Claude Code agents that default to them (e.g. Explore) must get an explicit model from this list, or the work goes to Luna/Terra instead. Cross-provider delegation is not an obligation.

At the edges (Luna, Fable) the assignment is binding; in the middle (Terra, Astra, Opus) the scopes overlap and the choice is judgement. Decide there by the *kind* of difficulty: unclear assignment - upwards; clear assignment with much routine - downwards; very large input - Opus.

A measured check (2026-09-16, `Werkzeuge/modell-bench`, 32 runs: two mid-sized coding tasks and
two multi-part logic tasks, every model twice) found no quality separation in that band: Luna,
Terra, Astra and Opus each passed 8 of 8 with zero defects. Opus was two to three times faster and
needed a third of the model turns, but cost 1.65 USD where the Codex models cost only allowance.
For work of that size the cheap route is therefore not a compromise, and speed is the only thing the
paid models buy. A second check the same day (18 runs: one contradiction-hunting task over a twelve-file record and
one open-ended concept task, Luna, Terra and Astra three times each) did separate them - but only
through blind review against fixed criteria, because the automated form checks again gave all
eighteen full marks. Opus judged the anonymised results without knowing the authors, and Astra took
the top three places in both tasks. The difference was substance, not style: only Astra traced the
faulty value through the code into the deletion statement, and only Astra kept concept and plan free
of contradictions. The single fabricated quotation in the field came from Luna. Terra and Luna did
not separate from each other. Astra paid for this with two to three times the wall-clock time and
roughly twice the input volume, so the scopes above hold as written: Astra where the difficulty is
real, Terra where the brief is clear, and no form check will tell the two apart.

Use local model listings and CLI status/help when accessible without launching a model task. Distinguish listed, authenticated, and proven runnable: none alone establishes the others. If access or the active model is unknown, make the recommendation conditional and explain what needs checking. Do not launch paid comparison calls just to choose a model.

For price-sensitive choices or comparative claims, consult current official [OpenAI model information](https://developers.openai.com/api/docs/models) and [Anthropic model information](https://platform.claude.com/docs/en/about-claude/models/overview). Check [Codex usage guidance](https://learn.chatgpt.com/docs/pricing) when using subscription allowances. API token prices are different from subscription usage; task costs also include context transfer, reasoning, retries, and host verification. The ranking above is a user preference, not a price record: no figures without a checkable source.

Both subscription allowances are measurable, so do not guess them. Before routing, read the
seven-day figure of each side:

- **Anthropic** (Opus, Fable): `week.pct` in `SecondBrain/index/usage-limits.json`, refreshed every
  15 minutes by a KI-OS poller from the same endpoint as `/usage`.
- **OpenAI** (Luna, Terra, Astra): `week.pct` in `~/.claude/statusline-openai-usage.json`, taken
  from the `rate_limits` the Codex CLI writes into its session logs. If the file is older than
  5 minutes, refresh it first with `node ~/.claude/statusline-openai-poll.mjs` (local files only,
  no network, no model call).

Trust a figure only when its `ts` is under 60 minutes old; a stale, missing or `null` figure means
unknown, not zero. The Claude Code status line shows both.

**Balance rule:** keep the two seven-day percentages as equal as possible. Compute the gap as
Anthropic minus OpenAI. Within 5 points the choice follows the scopes above alone. Beyond that,
the side that has used more gives way wherever the scopes overlap: with Anthropic ahead, clear
briefs go to Luna, Terra or Astra, and Opus and Fable keep only work that demonstrably needs them;
with OpenAI ahead, overlap work in the middle band (Terra/Astra vs. Opus) goes to Opus, and review
of Codex-written code goes to Claude. The larger the gap, the more decisively to shift. The
binding edges still hold - the balance never sends a Luna task to Fable or a large conception to
Luna. When one figure is unknown, route by scope only and say that the balance could not be
checked. The brief names both figures when they drove the choice.

## Return a short routing brief

Normally use fewer than 200 words:

- **Recommendation:** stay here or use a named provider/model for a specific role.
- **Why:** one or two reasons tied to this task, plus a material uncertainty if present.
- **Handoff:** the bounded assignment, relevant context/files, expected result, permitted actions, and how to check success. If staying here, give the immediate next step instead.

Offer at most one alternative when it helps a real tradeoff. Do not interview the user, generate a catalog of models, or start a plan-review loop. Do not claim to have changed the current session's model; a child CLI invocation is a separate session.

## Execute one handoff when requested

Use the selected provider's CLI from the correct project directory, explicitly selecting the recommended or requested model for that call. Check the actual binary's version and help; consult the relevant official [Codex non-interactive guide](https://learn.chatgpt.com/docs/non-interactive-mode) or [Claude programmatic guide](https://code.claude.com/docs/en/headless) if needed. Avoid changing global defaults, installing software, or switching models silently to make a call succeed.

Pass a self-contained brief with the goal, relevant requirements and files, constraints, expected output, and verification. For debugging, include failed attempts. For code inspection, identify the comparison baseline and relevant committed, staged, unstaged, and untracked changes. The child does not inherit the conversation. Send prompt text through stdin or a safely handled file; never interpolate arbitrary prompts into shell commands.

For review or diagnosis, use supported read-only project tools and restrict external write-capable tools too; a prompt saying "read-only" is not enforcement. If those restrictions cannot be established, return the prepared handoff and explain the limitation. For authorized edits, use scoped write permissions and preserve existing user changes; use an isolated worktree when concurrent edits would conflict. Never bypass permissions. The host pauses edits to the delegate's files while it works.

Use a fresh session for the one-off handoff, a bounded timeout, and separate stdout/stderr artifacts in a unique temporary directory outside the project. Keep the user informed during long calls. Read the completed result and exit status; an empty response, timeout, or permission failure is not success. Stop and report a failed handoff rather than automatically retrying, escalating to a larger model, or starting another round. If a timed-out process may still run, resolve its status before restarting work on the same files.

Assess findings against evidence, inspect any edits, and run appropriate checks within existing authorization. Report the outcome, checks actually run, and remaining uncertainty. Distinguish the model requested from the model observed; do not invent an observed identity. If a correction is outside the requested work, recommend it without expanding scope. Finish after this handoff and host verification; further delegation follows the user's request.
