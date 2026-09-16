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

## Select a practical candidate

Start with models available in the user's environment. The list below records the user's own routing preferences (Stand September 2026), not a permanent leaderboard. Rangfolge nach Kosten je Aufgabe: Luna < Terra < Astra < Opus < Fable.

- **GPT-5.6 Luna** (`gpt-5.6-luna`), günstigstes Modell: eng umrissene, wiederholbare Kleinarbeit mit maschinell prüfbarem Ergebnis — Fixtures, Extraktion, Doku-Updates, Syntax- und Dokumentationsfragen, kleine Design-Anpassungen (CSS-Klassen, Farben, Padding), isolierte Helfer. *Nicht,* wenn sich kein Abnahmekriterium formulieren lässt.
- **GPT-5.6 Terra** (`gpt-5.6-terra`): mittlere Coding-Arbeit, die mehr Urteil und Kontext braucht als eine Luna-Aufgabe — Standard-Backend (CRUD, REST-Endpunkte, SQL-Abfragen), Refactoring, Kommentare und Dokumentation. *Nicht* für Architekturentscheidungen oder Fehler, deren Ursache noch unbekannt ist.
- **GPT-6 Astra** (`gpt-6-astra`): schwierige, klar formulierte Aufträge — komplexe Algorithmen, Performance-Engpässe, mathematisch anspruchsvolle Logik, Analyse kryptischer Stack-Traces und tiefsitzender Logikfehler. Auch als unvoreingenommene zweite Meinung zu Architektur-Sackgassen. Von Claude Code aus der Kandidat für anbieterübergreifende Prüfung. *Nicht,* wenn der Auftrag selbst noch unklar ist oder die Eingabe das Kontextfenster sprengt.
- **Claude Opus** (`claude-opus-5`): schwierige Arbeit, deren Schwierigkeit in der *Menge* liegt statt in der Logik — Legacy-Codebases analysieren, riesige Logdateien zusammenfassen, hundertseitige Spezifikationen durcharbeiten, Architekturbewertungen über viele Dateien hinweg. Das Unterscheidungsmerkmal gegenüber Astra ist das Kontextfenster, nicht die Modellstärke; bei kleiner Eingabe ist Astra der günstigere Weg.
- **Claude Fable 5.1** (`claude-fable-5-1`), teuerstes Modell: Delegation und Konzeption — Mammut-Projekte in präzise Einzelaufträge für Luna und Terra zerlegen, komplexe Datenbankschemata, Sicherheitskonzepte und Produkt-Roadmaps entwerfen. Von Codex aus der Kandidat für anbieterübergreifende Prüfung. *Nicht* für Ausführung, die ein günstigeres Modell nach klarer Vorgabe erledigen kann.
- **Weitere verfügbare Modelle** (Sonnet, Haiku): bleiben Optionen, wenn Aufgabenzuschnitt, bereits vorhandener Kontext, Kontozugang oder eine ausdrückliche Nutzerwahl dafür sprechen. Anbieterübergreifende Delegation ist keine Pflicht.

Die Zuordnung an den Rändern (Luna, Fable) ist verbindlich; in der Mitte (Terra, Astra, Opus) überlappen die Zuschnitte und die Wahl liegt im Ermessen. Entscheide dort nach der *Art* der Schwierigkeit: unklarer Auftrag — aufwärts; klarer Auftrag mit viel Routine — abwärts; große Eingabemenge — Opus.

Use local model listings and CLI status/help when accessible without launching a model task. Distinguish listed, authenticated, and proven runnable: none alone establishes the others. If access or the active model is unknown, make the recommendation conditional and explain what needs checking. Do not launch paid comparison calls just to choose a model.

For price-sensitive choices or comparative claims, consult current official [OpenAI model information](https://developers.openai.com/api/docs/models) and [Anthropic model information](https://platform.claude.com/docs/en/about-claude/models/overview). Check [Codex usage guidance](https://learn.chatgpt.com/docs/pricing) when using subscription allowances. API token prices are different from subscription usage; task costs also include context transfer, reasoning, retries, and host verification. Die Rangfolge oben ist eine Nutzervorgabe, kein Preisbeleg: keine Zahlenangaben ohne prüfbare Quelle.

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
