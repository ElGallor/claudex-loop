# debatte-v1: rules for a discussion round

Read this before running a discussion round, and hand it to any subagent that runs one for you. It is the chat-side form of the contract the Denk-Cockpit debate engine implements (`Werkzeuge/denk-cockpit/lib/debatte*.js`, Dok. 19 section 3.5, Dok. 20); the rules are the same, only the moderator differs: there it is code, here it is the host session.

## What it is for

Two equal sides examine one decision and either reach the same option or hand the user a clean dissent. Use it for judgements: an architecture or scope choice, a trade-off, the reading of a failed gate or test, a large follow-up order. Do not use it for anything that can be measured or looked up, for rules, or for rights that belong to the user. A measured three-round debate cost 3.86 USD on the Opus side and 175,140 tokens on the Astra side (2026-09-23), so a small reversible choice is decided by the host, and a plain check of one plan or one diff is a second opinion, not a debate.

## Setup

- **Question:** one written question and a decision template (`vorlage`) on disk. Record its path and SHA256.
- **Options:** `O0`, `O1`, ... with a title each. `O0` is always the null option ("change nothing") and must be examined seriously.
- **Dossier:** the files both sides may read, as paths. Both sides get the identical list.
- **Mode:** `wahl` (choose one of the listed options) or `gestaltung` (develop a proposal; each side lists its points of contention with the same `thema` wording across rounds).
- **Sides:** A = `claude-opus-5-5` at effort `high`, B = `gpt-6-astra` at effort `high`. The sides are equal. The moderator (host, or a Sonnet or Opus subagent) has no vote, writes no move of its own and does not summarise one side to the other: each side reads the other's move verbatim from its file.
- **Rounds:** `runden_max` is 2 or 3. Three is the default; two when the allowance is tight.

## Rounds

1. **Round 1, blind and parallel.** Both sides answer independently without seeing the other.
2. **Round 2.** Each side receives the path of the other side's round-1 move, answers its objections and carries its own objections forward, marking each `offen`, `aufgeloest` or `zurueckgezogen`.
3. **Round 3 only without agreement** after round 2.
4. **Confirmation move** when dissent remains after the last round: each side is shown the other's final position and answers `annehmen` or `ablehnen` with one sentence of reason. No reading tools in this move.

After every round the moderator checks agreement mechanically:

- **Agreement (`einig`)** = both sides stand on the same option **and** neither holds an open objection of severity `hoch`. Rejecting an offer in an earlier move does not undo an agreement if the other side accepts the counter-offer in the end.
- **One-sided acceptance** in the confirmation move: the accepted rule of the other side holds (`einigung_ueber: annahme_gegenangebot`).
- **Both accept different options:** dissent stays. Sides must not swap positions crosswise just to force agreement; a position is taken over only with evidence.
- **Dissent (`dissens`)** goes to the user as a decision: both final positions, the open `hoch` objections, and what each side conceded. Never manufacture convergence, never let the moderator break the tie.
- Agreement on a deep intervention still needs the user's release; the debate prepares the decision, it does not replace it.

## The move

Every content move is one JSON object following [debatte-zug-v1.json](debatte-zug-v1.json):

| Field | Content |
|---|---|
| `option` | the option id the side stands on (`O0`, `O1`, ...) |
| `konfidenz` | 0 to 1; information only, never a deciding quantity |
| `begruendung` | why this option |
| `belege` | evidence as `file:line` or unit id; no evidence, no claim |
| `einwaende` | list of `{id, schwere: hoch/mittel/gering, gegen: option, beleg, aufloesung: offen/aufgeloest/zurueckgezogen}` |
| `zugestaendnisse` | what the side concedes to the other |
| `offen` | questions neither side could settle |
| `pre_mortem` | how the chosen option fails; mandatory (not null) for interventions of depth 6 and above |
| `markdown` | the readable version of the move |

## Briefs

The side briefs are German, because the protocol and the user's files are. Preamble for every content move:

```
Zwei gleichrangige Seiten pruefen eine Frage; der Moderator hat keine Stimme.
Antworte ausschliesslich im vorgegebenen JSON-Schema. Lesbarer Teil ins Feld markdown, keine Vorrede.
Nur lesen, nichts aendern. Belege als Datei:Zeile angeben.
Die Nulloption O0 ist eine ernsthaft zu pruefende Option.
Lies zielgerichtet nur, was das Argument traegt: Lesen kostet. Dateien werden per Pfad verwiesen.
Uebernimm eine Position nur mit Beleg, nicht allein deshalb, weil die Gegenseite sie vertritt.
konfidenz ist Auskunft, keine Entscheidungsgroesse.
```

Round 1 adds: the question, the mode, the path of the decision template, the dossier paths, the option list, and "Waehle genau eine Option-ID; begruende die Wahl und pruefe O0 ernsthaft. Nenne tragende Belege, Einwaende mit Schwere und aufloesung, Zugestaendnisse und offene Fragen."

Later rounds add: "Runde N: pruefe und entwickle deine Position weiter. Letzter Zug der Gegenseite: <Pfad>. Beantworte die Einwaende der Gegenseite und schreibe eigene Einwaende mit aufloesung fort. Kennzeichne aufgeloeste oder zurueckgezogene Einwaende; begruende Aenderungen mit Belegen. Uebernimm keine Gegenposition ueberkreuz, nur um Einigung zu erzwingen."

## Calls

Each side is a separate process. Prompt through stdin from a file, one artefact directory per move outside the project, a 600 s limit per move.

- **Opus side:** `claude -p --model claude-opus-5-5 --effort high --output-format json --safe-mode --tools Read,Glob,Grep --allowedTools Read,Glob,Grep --permission-mode dontAsk --permission-prompts none --json-schema <schema> --add-dir <each read root>`; later rounds add `--resume <session id of this side's previous successful move>`. The confirmation move runs with `--tools ""`.
- **Astra side:** `codex exec -m gpt-6-astra -c model_reasoning_effort="high" -s read-only -c approval_policy="never" --json --skip-git-repo-check --output-schema <schema file> -o <answer file>`, with stdin closed after the prompt (`< prompt.txt`; a background run with open stdin hangs). Later rounds use `codex exec resume <session id> ...`. Needs Codex CLI 0.156.1 or newer.
- Check the actual binary's help before the first call; flags move between versions.

## Failures

A timeout, a schema violation, an empty answer or a non-zero exit is a technical failure, never a position. Restart the same side once with the same model and effort. If it fails again, the debate ends `abgebrochen` and the user gets that result. Never fall back to another model, never resume from a failed move, never read silence as consent.

## Record

Write one protocol file per debate: question, template path and hash, options, sides with model requested and model observed, effort and session ids, every move verbatim per round, outcome (`einig`, `dissens`, `abgebrochen`), how agreement was reached (`runde` or `annahme_gegenangebot`), the decision or the dissent handed to the user, and the measured cost (Opus `total_cost_usd`; Astra tokens, since the Codex CLI shows no credits per call).

## Second opinion: the one-round form

A second opinion uses the same setup and the same move schema but only the other side and only round 1: from Claude Code one Astra move at `high`, from Codex one Opus move at `high`. The host states its own position and the criteria before it reads the answer, then arbitrates each objection against evidence. An objection of severity `hoch` that the host cannot resolve with evidence turns the second opinion into a discussion round or goes to the user.
