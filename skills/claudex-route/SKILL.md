---
name: claudex-route
description: "Recommend a model and a scoped handoff for a task in Claude Code or Codex, and obtain second opinions or run a discussion round on a decision. Use when choosing who should handle a task, weighing a decision, seeking a second opinion, getting unstuck, or delegating focused work; execute one handoff or one discussion round when requested."
---

# Claudex Route

Choose a useful next step for the task: keep it with the current agent, obtain a second opinion, run a discussion round on a decision, investigate a blocker, or delegate a bounded piece of work. Give a brief recommendation before any requested execution. This skill is self-contained and is the single routing, second-opinion and discussion skill in this environment; the other skills of this plugin (claudex-loop, codex-build, codex-review) are deactivated since 2026-09-30.

## Understand the job

Use the current conversation and just enough relevant project context to identify the host, current model when known, desired outcome, and constraints. Consider ambiguity, code dependencies, verification difficulty, context size, and cost or speed preferences. A short prompt can describe a difficult task. Preserve explicit model and provider choices; do not silently replace them with your preferred pairing.

Treat a request for advice as recommendation-only. A request to route and perform work authorizes the scoped handoff within the user's existing permissions. The skill invocation by itself does not authorize implementation. Ask a question only when a missing fact materially changes the recommendation or authorized work; otherwise state your assumption.

## Choose the role before the model

| Situation | Useful next step |
|---|---|
| Straightforward task, sufficient context, no reason to delegate | Stay with the current agent only within its model scope; an Opus host delegates all building |
| A consequential or ambiguous plan is ready | Second opinion: ask the other provider to challenge requirements, assumptions, and acceptance criteria before building |
| An implementation is ready | Second opinion: ask the other provider to inspect relevant changes against requirements, including test validity |
| Repeated attempts have failed | Give another provider the reproduction, evidence, and failed approaches; ask for a testable alternative explanation |
| A separable task has clear inputs and acceptance checks | Delegate that piece to a suitable smaller model and inspect its result |
| A decision with real options whose wrong choice is expensive to undo - architecture, scope, a trade-off, the reading of a failure | Run a discussion round between Opus and Astra and bring agreement or dissent to the user |

The current agent keeps the user's requirements and coordinates the work. A different provider offers another perspective, not guaranteed correctness. A cheaper delegate does not need to be a peer reviewer of the entire architecture.

Delegate by default and treat doing the work yourself as the exception that needs a reason: the host hands out bounded assignments with clear inputs and acceptance checks. An Opus host does not build; it writes the brief, delegates and accepts the result. A clear build goes to Sol as one undivided brief, and deep debugging goes whole to Astra low. The limit is specification cost. Writing a precise assignment is itself work, so delegation pays only when the task is substantially larger than the brief that describes it; splitting small work costs more than it saves. This limits splitting, not the rule that Opus delegates building.

Prefer a reviewer who did not write the work. Independence is a gradient, not a switch - same session, then a fresh session of the same model, then a different model, then a different provider - so move as far along it as is practical, and at minimum use a fresh session. The reason is plausible but unproven: a model tends to re-read its own work with the assumptions it wrote it with. Treat it as a precaution, not an established fact, and the closer reviewer and author stand, the more the review must hang on external criteria fixed in advance.

## Select a practical candidate

Start with models available in the user's environment. The list below records the user's own routing preferences (September 2026), not a permanent leaderboard. By cost per task: Luna < Sol < Sonnet < Opus < Fable; Astra belongs above Sol, measured at four to five times Sol 6 medium for the build below, or roughly six times Sol 6.1 low (report 24, section 4.3). These measurements do not place Astra against the Claude models. Luna, Sol and Astra run through the Codex CLI on the OpenAI subscription allowance; Sonnet, Opus and Fable draw on the Anthropic allowance. GPT-6 Luna and the historical GPT-6 Sol bench anchor need Codex CLI 0.156.1 or newer - older versions fail with HTTP 400 "model is not supported when using Codex with a ChatGPT account" (checked 2026-09-23). GPT-6.1 Sol needs Codex CLI 0.159.2 or newer. The standard in `~/.codex/config.toml` is `gpt-6.1-sol`/`low`; always select the model explicitly with `-m` and the effort with `-c model_reasoning_effort="..."`. The GPT-5.6 models remain callable but are superseded here. Each entry gives a short scope and then what the model should not be used for - exclusions age better than task lists and also decide cases nobody listed.

- **GPT-6 Luna** (`gpt-6-luna`), cheapest: narrow, repeatable work with a machine-checkable result - fixtures, extraction, doc updates, small isolated helpers - and checks a machine can decide: a test run, schema validity, completeness against a given list, diff size. *Never* for judgement checks such as "is this code good" or "is the architecture sound": a "looks fine" from a model too weak to judge reads like an approval and is worse than no review at all. *Not* when no acceptance criterion can be stated. Effort `high`: at `low` and `medium` it diagnosed a fix task correctly but stopped before editing in 3 of 4 runs ("shall I apply the fix?"); at `high` it finished 8 of 8.
- **Sol** (`gpt-6.1-sol` for all Sol work, user decision 2026-09-30): the workhorse and default builder. Clearly stated changes and implementation go to it as **one undivided brief**. **All bulk and diligence work goes to Sol, whoever coordinates** - reading and sweeping code, greps and inventories, index building and file summaries, routine backend work, refactoring, comments and documentation, report drafts, and standard coding that needs more judgement and context than a Luna task. A Sonnet or Opus coordinator hands this work to Sol and does not do it itself because it happens to be at hand: per task Sol is the cheapest capable route (see the price comparison below). Sol has no allowance threshold. *Not* for architecture decisions, and *not* for defects whose cause is still unknown. Effort `low` for builds, bulk and diligence work, documents, plans and analyses. Use `medium` only where the caller has measured it for that job, as in the user's Kartograf runs; it is not the document default. `gpt-6.1-sol` replaces `gpt-6-sol` in all current Sol assignments.
- **Claude Sonnet 5.5** (`claude-sonnet-5-5`, alias `sonnet`): clearly stated assignments only where a Claude model is technically required - agent frontmatter, Claude subagents and workflow agents. *Not* for finding a cause, tracing a value through code, or any review or acceptance judgement: on the contradiction-hunting task it ranked below Sol, never stated that the code ignores the configuration, and one run invented line numbers. *Not* for very large reading loads: at about 2.9M input tokens it was measured as more expensive than Opus. *Not* for the bulk itself, which is Sol's. Effort `low` for clear coding (16 of 16, no difference to `medium`), `high` for multi-part documents and plans, where `medium` lost values between concept and plan.
- **Claude Opus 5.5** (`claude-opus-5-5`), the strongest model in this environment: delegator, acceptor and debate partner - distributes work, writes briefs, accepts results and makes judgements. It is one side of every discussion round. **It does not build.** *Not* for hard builds or deep debugging, which go to Astra low; an assignment that is hard only because it is vague is clarified first, then goes to Sol. *Not* for bulk, which is Sol's - the allowance spent there is missing from the work that needs it. *Not* on repository size alone, which does not prove a need for that much context at once. *Not* for condensing a large input per task so it fits a smaller window - reading the full context is exactly the expensive part, so the saving is cancelled; only an index built once and reused is worth it, and building it belongs to Sol or Luna. Effort `high`.
- **GPT-6 Astra** (`gpt-6-astra`): hard builds - unknown cause, deep debugging taken whole instead of split, demanding logic and tracing a value through code; and the end review of risky builds, once at the end, read-only. Risky means code close to money (live trading, capital, risk), gate and security code, and field/interface renames with silent consumers. It is also the other provider's strongest voice for discussion rounds and second opinions (section below). *Not* for clear builds, which are Sol's. *Not* for an end review on every build. *Not* as the routine coordinator of sub-workers (see below). Effort `low` for hard builds and risky end reviews, `high` for discussion rounds and second opinions.
- **Claude Fable 5.1** (`claude-fable-5-1`), most expensive: conception and decomposition - cutting large undertakings into precise assignments, designing schemas, security concepts and roadmaps. From Codex it is the candidate for a second opinion on conception. *Not* for execution once a design is agreed: from there on the work is derivation, and a cheaper model does it from the stated brief.
- **Haiku is not used** in this environment. Built-in Claude Code agents that default to it (e.g. Explore) get an explicit model: `sonnet` for a clearly scoped search, or the sweep goes to Sol or Luna instead. Cross-provider delegation is not an obligation.

- **Do not use Sol at `high`.** In report 24, sections 4.2 and 4.3, Sol 6 high cost 12-15 credits and took 15.1-17.6 min against medium at 7-9 credits and 11.2-12.1 min: roughly 1.5 times the cost and 1.4 times the duration, with the same 52/52 tests; low passed them too.
- **Do not use Astra as the routine coordinator of sub-workers.** It is technically possible: `spawn_agent` is built in and works in `workspace-write`; nested `codex exec` through the shell worked only with `danger-full-access`, which is not needed for the built-in route (report 24, section 3). Coordination cost 65-66 credits with Sol 6 low and Luna high, or 60 with Sol 6.1 low, against 35-38 for Astra medium alone - roughly twice as much, with no time gain (sections 4.2 and 4.3). Against one undivided brief to Sol it saved no Opus effort in the comparison of briefs and reports; actual Anthropic consumption was not measured (section 5).
- **Superseded or removed:** `gpt-6-sol` is superseded by `gpt-6.1-sol` for all Sol work and remains only as a bench anchor and in historical measurements. GPT-5.6 Terra/Luna and `gpt-5.6-sol` are superseded (Terra by Sol, Luna by `gpt-6-luna`); DeepSeek is removed. Haiku is never used.

Choose in this order for each work slot:

1. Must it technically be a Claude model (agent frontmatter, Claude workflow)? Use `sonnet`, unless the work is cause-finding, judgement or close to money - then `opus`.
2. Otherwise use Codex: clearly stated builds, diligence work, documents, plans and analyses go to Sol 6.1 `low` (`medium` only when measured by the caller for that job); small, machine-checkable work goes to Luna `high`; genuinely hard work goes to Astra `low`.
3. If the assignment is hard only because it is vague, clarify it first - goal, inputs as paths, steps, output format and machine-checkable acceptance - then give it to Sol.
4. Opus stands at no build slot any more. It stays where work is delegated, accepted or judged - which includes the Claude-only slots of step 1 whose work is cause-finding, judgement or close to money.

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
real, Sol as Terra's successor where the brief is clear, and no form check will tell the two apart.

Both checks ran on GPT-5.6 Luna and Terra. A third check (2026-09-23, 84 runs, GPT-6 Luna and Sol
at `low`/`medium`/`high`, Astra at all three, blind review of the S4/S5 documents with the
2026-09-16 Astra and Terra results mixed in as anchors) placed the successors: the anchors came out
as before (old Astra ranks 1-3, old Terra near the bottom), Astra led at every effort level, Sol
sat clearly above old Terra and below Astra, and Luna took the last places on the judgement tasks.
On clear coding (S2/S3) every configuration passed except Luna at `low`/`medium`, which stopped
before editing. Two runs per configuration - read it as a direction, not a ranking to the decimal.
Since the afternoon decision on 2026-09-30, Astra takes genuinely hard builds at `low`,
while clearly stated builds go to Sol; discussion rounds and second opinions stay at `high`.

A fourth check (2026-09-30, 24 runs, Sonnet 5.5) placed the new model. On clear coding (S2/S3) it
passed 16 of 16 at `low` and `medium` and was the fastest model measured (13-24 s). On the
judgement tasks two Opus reviewers ranked its documents blind among Astra, Sol, Luna and old Terra
anchors, which again came out as before. Contradiction hunting (S4): Sonnet took mean rank 10.75
(`medium`) and 12.25 (`high`) of 15, below Sol (6.5) and level with Luna and old Terra - none of
its four documents stated that the code never reads the configuration, one invented three line
numbers, one misquoted the changelog. Concept and plan (S5): `high` reached mean rank 5.0, between
Astra and Sol `medium`; `medium` fell to 11.25, level with Sol `low`. Opus 5.5 itself was not
placed on S4/S5 - it is treated as the strongest model by the user's decision, not by this bench.

A fifth check (2026-09-30, `Denk-Cockpit/24-Astra-als-Bauer-und-Delegator-2026-09-30.md`,
`lagerwerk`, 23 runs: a multi-part build with silent consumers and rules to recover from code)
compared builders and delegation. Spans are over the runs; n = 1-3 means direction, not a
ranking to the decimal:

| Variant | n | Hidden tests | Duration | Credits |
|---|---|---|---|---|
| B1 Astra low alone | 3 | 52/52 x3 | 5.7-6.3 min | 30-34 |
| B2 Astra medium alone | 3 | 52/52 x3 | 7.0-7.7 min | 35-38 |
| B9 Sol 6 low alone (additional check) | 2 | 52/52 x2 | 7.3-9.6 min | 6-7 |
| B3 Sol 6 medium alone | 3 | 52/52 x3 | 11.2-12.1 min | 7-9 |
| B4 Sol 6 high alone | 3 | 52/52 x3 | 15.1-17.6 min | 12-15 |
| B7 Sol 6.1 low alone | 2 | 52/52 x2 | 7.9-8.3 min | 4.5-5.4 |
| B8 Sol 6.1 medium alone | 2 | 52/52 x2 | 10.0-12.0 min | 5.5-6.9 |
| B5 Astra medium coordinates Sol 6 low + Luna high | 2 | 52/52 x2 | 7.6-8.7 min | 65-66 |
| B5' Astra medium coordinates Sol 6.1 low + Luna high | 1 | 52/52 | 7.2 min | 60 |
| B6 External split: Sol medium (parts 1-2), Sol low (parts 3-5), Luna high (part 6), Astra low end review | 2 | 52/52 x2; before review: 52/52 and 51/52 | 17.5-18 min (sequential) | 25-27 |

The task has a ceiling: every variant reached 52/52 - it separates by price and time, not quality;
it does not establish parity on unknown causes or genuinely hard work, and code quality was not
reviewed blind (report 24, section 4.5).
In one B6 run Astra's end review found a real rule error by Sol in the split assignment - the
exemption of services when reserving, which Sol had right in all twelve undivided runs - a
single-run finding, not a structural result.

Credit rates per 1M input / cached / output tokens as used in report 24: Sol 6 50 / 5 / 250,
Astra 250 / 25 / 1250, Luna 2.5 / 0.25 / 12.5; **Sol 6.1 provisionally 50 / 2.5 / 250**,
from a retrieval of the [pricing page](https://learn.chatgpt.com/docs/pricing) through a summarising
tool, with no date on the page. The historical Sol figures in the price comparison below refer to Sol 6.

A sixth check (2026-09-30, `Denk-Cockpit/25-Bench-Sol-6-1-S4-S5-2026-09-30.md`,
blind review of S4/S5, n=2 per cell) supports the user's decision to use Sol 6.1 `low`
for documents, plans and analyses too. Mean ranks are out of 19 (lower is better);
credits are means per run, shown separately for each task:

| Variant | Mean rank S4 / S5 | Credits per run S4 / S5 |
|---|---|---|
| Sol 6.1 medium | 2.5 / 7.0 | 3.91 / 3.51 |
| Sol 6.1 low | 7.0 / 5.75 | 2.66 / 2.60 |
| Sol 6 medium (bench anchor) | 10.5 / 10.25 | 1.97 / 2.57 |
| Sol 6 low (bench anchor) | 4.5 / 17.25 | 2.25 / 1.73 |

On the S4 core criterion, all 4 of 4 Sol 6.1 documents trace the value through the code,
with 0 invented citations; Sol 6 medium received full marks in 1 of 4 judge votes.
Sol 6.1 low costs about the same as Sol 6 medium and ranked ahead of it in S4 and S5;
Sol 6.1 medium costs 37-98 % more credits than Sol 6 medium and takes 2.7-3 times as long.
The judge spread is larger than the gap, so "better than Sol 6" is not proven. S4 calibration
only partially passed (Astra was not at the top), so S4 ranks against Astra are not reliable.
Sol 6.1 sets more starting values of its own in S5 (8-9 per cell against 5 for Sol 6 medium).
The Sol 6.1 credit rate is provisional; durations were not isolated from parallel machine load.

Price comparison Sonnet against Sol (fourth check, mean per run). Sonnet in USD as billed by the
CLI (2 / 10 per 1M input / output, cache read 0.2, cache write 4). Sol in Codex credits from
tokens times the published rate (50 / 5 / 250 per 1M input / cached / output); the last column
converts at 25 credits per USD, which is the ratio between the credit rates and the API list
prices and not a price anyone is charged on the subscription:

| Task | Sonnet 5.5 | Sol | Sol at list price |
|---|---|---|---|
| S2/S3 clear coding | `low` 0.092 USD, 15 s | `low` 0.96 credits, 41 s | 0.038 USD |
| S4 contradiction hunt | `medium` 0.224 USD, 39 s | `medium` 2.30 credits, 67 s | 0.092 USD |
| S5 concept and plan | `high` 0.375 USD, 150 s | `medium` 2.85 credits, 136 s | 0.114 USD |

List prices per token are the same; per task Sonnet costs two to three times as much, because it
pays for writing its start context into the cache on every fresh run and produces more output.
Sonnet is two to three times faster on the first two. Hence the split: bulk and clear builds
to Sol, Sonnet only where a Claude model is technically required and the assignment is clear.

Use local model listings and CLI status/help when accessible without launching a model task. Distinguish listed, authenticated, and proven runnable: none alone establishes the others. If access or the active model is unknown, make the recommendation conditional and explain what needs checking. Do not launch paid comparison calls just to choose a model.

For price-sensitive choices or comparative claims, consult current official [OpenAI model information](https://developers.openai.com/api/docs/models) and [Anthropic model information](https://platform.claude.com/docs/en/about-claude/models/overview). Check [Codex usage guidance](https://learn.chatgpt.com/docs/pricing) when using subscription allowances. API token prices are different from subscription usage; task costs also include context transfer, reasoning, retries, and host verification. The ranking above is a user preference, not a price record: no figures without a checkable source.

Both subscription allowances are measurable, so do not guess them. Before routing, read the
seven-day figure of each side:

- **Anthropic** (Sonnet, Opus, Fable): `week.pct` in `SecondBrain/index/usage-limits.json`, refreshed every
  15 minutes by a KI-OS poller from the same endpoint as `/usage`.
- **OpenAI** (Luna, Sol, Astra): `week.pct` in `~/.claude/statusline-openai-usage.json`, taken
  from the `rate_limits` the Codex CLI writes into its session logs. If the file is older than
  5 minutes, refresh it first with `node ~/.claude/statusline-openai-poll.mjs` (local files only,
  no network, no model call).

Trust a figure only when its `ts` is under 60 minutes old; a stale, missing or `null` figure means
unknown, not zero. The Claude Code status line shows both.

**Balance rule:** keep the two seven-day percentages as equal as possible. Compute the gap as
Anthropic minus OpenAI. Within 5 points the choice follows the scopes above alone. Beyond that,
the side that has used more gives way wherever the scopes overlap: with Anthropic ahead, Opus
and Fable keep only work that demonstrably needs them; Sonnet stays only where a Claude model is
technically required. With OpenAI ahead, Astra is restricted to work that really meets its
definitions: hard builds, end reviews of risky builds, and real decisions. **Sol is exempt
from the balance and has no threshold:** bulk and diligence work always goes to Sol, and whatever
Sol can carry goes to Sol as often as possible, at any OpenAI figure - the user sizes that
subscription for it (decision 2026-09-30). The larger the gap, the more decisively to shift. The binding edges still
hold - the balance never sends a Luna task to Fable, a large conception to Luna, or build work
back to Opus. When one figure is unknown, route by scope only and say that the balance could not be
checked. The brief names both figures when they drove the choice.

## Second opinions and discussion rounds

Weighing, open questions and decisions come up in every assignment, and handling them is a fixed part of the work, not an exception: whenever a choice has real options and the wrong one is expensive to undo, do not settle it alone. Two forms, both read-only and both between providers:

- **Second opinion** - one round. A fresh session of the other provider's strongest model (from Claude Code: Astra at `high`; from Codex: Opus at `high`, Fable for conception) gets the question, the options, the evidence as file paths and the criteria fixed in advance, and answers in the move schema. Use it for a plan before building, a finished implementation before acceptance, a diagnosis before an expensive fix. The host states its own position first, then arbitrates each objection against evidence. A second opinion is another perspective, not an approval.
- **Discussion round** - up to three rounds under the `debatte-v1` rules: Opus at `high` against Astra at `high` as equal sides, round 1 blind and parallel, round 2 with sight of the other's move, round 3 only without agreement, then a confirmation move. Agreement means the same option and no open objection of severity `hoch`. Dissent goes to the user with both final positions; never manufacture convergence. A technical failure gets one restart with the same model and is never read as consent.

Before running either form, read the [debatte-v1 reference](references/debatte-v1.md): setup, move schema ([debatte-zug-v1.json](references/debatte-zug-v1.json)), brief texts, CLI calls, failure rules and the record to write. When a subagent or coordinator runs the round, hand it that reference file by path rather than retelling it. The moderator - host, or a Sonnet or Opus subagent - has no vote and does not paraphrase one side to the other.

A three-round debate was measured at 3.86 USD on the Opus side plus 175,140 Astra tokens, so keep the forms apart: what can be measured or looked up is measured, a small reversible choice is decided by the host and stated as an assumption, a single plan or diff gets a second opinion, and only a real decision gets a discussion round. Rules and rights that belong to the user are never debated. An author does not give the second opinion on its own work.

## Return a short routing brief

Normally use fewer than 200 words:

- **Recommendation:** stay here, use a named provider/model for a specific role, or take the decision to a second opinion or discussion round.
- **Why:** one or two reasons tied to this task, plus a material uncertainty if present.
- **Handoff:** the bounded assignment, relevant context/files, expected result, permitted actions, and how to check success. If staying here, give the immediate next step instead.

Offer at most one alternative when it helps a real tradeoff. Do not interview the user or generate a catalog of models. Do not claim to have changed the current session's model; a child CLI invocation is a separate session.

## Execute one handoff when requested

Use the selected provider's CLI from the correct project directory, explicitly selecting the recommended or requested model for that call. For Codex always set the model with `-m` and the effort with `-c model_reasoning_effort="..."`, even when they match the `gpt-6.1-sol`/`low` standard in `~/.codex/config.toml`. Sol 6.1, used for all Sol work, needs CLI 0.159.2 or newer; GPT-6 Luna and the historical Sol 6 bench anchor need 0.156.1 or newer. Check the actual binary's version and help; consult the relevant official [Codex non-interactive guide](https://learn.chatgpt.com/docs/non-interactive-mode) or [Claude programmatic guide](https://code.claude.com/docs/en/headless) if needed. Avoid changing global defaults, installing software, or switching models silently to make a call succeed.

Pass a self-contained brief with the goal, relevant requirements and files, constraints, expected output, and verification. For debugging, include failed attempts. For code inspection, identify the comparison baseline and relevant committed, staged, unstaged, and untracked changes. The child does not inherit the conversation. Send prompt text through stdin or a safely handled file; never interpolate arbitrary prompts into shell commands.

For review or diagnosis, use supported read-only project tools and restrict external write-capable tools too; a prompt saying "read-only" is not enforcement. If those restrictions cannot be established, return the prepared handoff and explain the limitation. For authorized edits, use scoped write permissions and preserve existing user changes; use an isolated worktree when concurrent edits would conflict. Never bypass permissions. The host pauses edits to the delegate's files while it works.

Use a fresh session for the one-off handoff, a bounded timeout, and separate stdout/stderr artifacts in a unique temporary directory outside the project. Keep the user informed during long calls. Read the completed result and exit status; an empty response, timeout, or permission failure is not success. Stop and report a failed handoff rather than automatically retrying, escalating to a larger model, or starting another round. If a timed-out process may still run, resolve its status before restarting work on the same files.

Assess findings against evidence, inspect any edits, and run appropriate checks within existing authorization. Report the outcome, checks actually run, and remaining uncertainty. Distinguish the model requested from the model observed; do not invent an observed identity. If a correction is outside the requested work, recommend it without expanding scope. Finish after this handoff and host verification; further delegation follows the user's request.
