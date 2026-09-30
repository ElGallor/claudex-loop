# Measurements behind the routing scopes

Evidence for the scopes in `SKILL.md`. Read it when a choice is contested or a figure is needed; routing itself does not need it. Older model names (Sol 6, GPT-5.6 Terra/Luna) appear here only as bench anchors.

## Exclusions with their measurements

- **Do not use Sol at `high`.** In report 24, sections 4.2 and 4.3, Sol 6 high cost 12-15 credits and took 15.1-17.6 min against medium at 7-9 credits and 11.2-12.1 min: roughly 1.5 times the cost and 1.4 times the duration, with the same 52/52 tests; low passed them too.
- **Do not use Astra as the routine coordinator of sub-workers.** It is technically possible: `spawn_agent` is built in and works in `workspace-write`; nested `codex exec` through the shell worked only with `danger-full-access`, which is not needed for the built-in route (report 24, section 3). Coordination cost 65-66 credits with Sol 6 low and Luna high, or 60 with Sol 6.1 low, against 35-38 for Astra medium alone - roughly twice as much, with no time gain (sections 4.2 and 4.3). Against one undivided brief to Sol it saved no Opus effort in the comparison of briefs and reports; actual Anthropic consumption was not measured (section 5).

## Bench checks

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

## Price comparison Sonnet against Sol

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

## CLI versions

GPT-6.1 Sol needs Codex CLI 0.159.2 or newer; GPT-6 Luna needs 0.156.1 or newer. Older versions fail with HTTP 400 "model is not supported when using Codex with a ChatGPT account".
