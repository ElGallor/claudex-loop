# Availability, prices and the allowance balance

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
- **MiniMax** (Flash): `GET https://api.minimax.io/v1/api/openplatform/coding_plan/remains` with
  `Authorization: Bearer $MINIMAX_API_KEY` (never print the key); entry `model_name: general`,
  fields `current_interval_remaining_percent` (5-hour window) and `current_weekly_remaining_percent`.
  These are percent *remaining*, not used. Compare with Sol's figure only to pick between the two.

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
