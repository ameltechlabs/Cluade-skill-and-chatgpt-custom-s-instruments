# AmelTech Chat Efficiency

An adaptive reasoning and token-efficiency skill for Claude. It sizes its reasoning to the task, removes waste without removing substance, and never fabricates token numbers.

> **Less wasted reasoning, not less useful reasoning.**

## Structure

```text
ameltech-chat-efficiency/
├── SKILL.md                          Router + complete Tier 0/1 procedure (always loaded when triggered)
├── README.md                         This file (for humans; Claude doesn't need it)
├── references/
│   ├── reasoning-council-v7.md       Tier 2 council: P1–P7, adjudication, cost gate, output formats, loop limits
│   ├── token-optimization.md         Deletion test, compression patterns, user controls, Delta mode
│   ├── telemetry-spec.md             Token definitions, states, source rules, dashboards, tracking
│   └── workflow-engine.md            Dependency graphs, interference scan, critical path, rollback, ACT execution
└── scripts/
    └── telemetry.py                  Dashboard calculator (Python 3, standard library only)
```

Each rule is stated in exactly one file; the others refer to it. Claude reads a reference only when its trigger applies, so simple questions never load the council.

## How it works

**Three tiers**, chosen by an ordered test (first match wins):

| Tier | When | Reasoning |
|---|---|---|
| 0 — Direct | One defensible answer, low risk | Answer (with a freshness check if the fact can change) |
| 1 — Light council | A judgment, low stakes | P1 → P2 → P6 |
| 2 — Full council | Any *material* trigger: high stakes, irreversible, contested, complex multi-step… | All seven pillars |

**Tier 2 modes:** ANALYZE (decisions and claims → next actions), PLAN (you execute → workflow + rollback), ACT (Claude executes with tools → checkpoints + confirmation before irreversible steps).

**The seven pillars:** P1 First Principles · P2 Contrarian · P3 Outsider · P4 Expansionist (stress-test) · P5 Executor · P6 Synthesizer (answer + HIGH/MEDIUM/LOW confidence) · P7 Verifier (external checks only).

**Evidence levels (E1–E5):** from direct checks (E1) down to assumptions (E5). Competing answers are decided by the stronger evidence, confidence is capped by the weakest key fact, and only E1–E3 counts as verification.

**Deletion test:** a sentence, bullet, table row or comment stays only if removing it would lose something you need, reduce your ability to act or verify, or change your decision. Safety, rollback and uncertainty information is never cut.

**Compound messages:** several questions in one message are split and each gets its own tier, so one hard question doesn't make the easy ones verbose.

**Delta mode:** follow-up messages reopen only the affected pillar and return only what changed. Replies to Claude's own clarifying question resume the paused task instead.

## Using it

The skill loads when a request matches its description — e.g., "efficient mode", "run the council", "show council", "how many tokens did that use", or a complex/high-stakes plan.

| Say | Effect |
|---|---|
| "brief" / "just the answer" | Answer + confidence + essential warnings only |
| "full" / "explain" / "show council" | Adds explanation, evidence and per-pillar notes |
| "tier 0" / "tier 1" / "tier 2" | Pins the reasoning depth for one request |
| "tokens" / "dashboard" | Shows the token dashboard |
| "track tokens" / "stop tracking" | Turns savings tracking on/off |

**Always-on use:** skills load on demand, not on every message. To apply this behavior to every reply, add a short summary of it to your user preferences or a Project's instructions.

## Token telemetry

Every figure is labeled **MEASURED**, **ESTIMATED** `(est.)`, or the dashboard says **UNAVAILABLE**. In chat apps Claude cannot see its own token counts, so the default is UNAVAILABLE. Savings are computed only against a real baseline, and only when baseline and result come from the same kind of source.

```bash
python scripts/telemetry.py --input-tokens 12430 --output-tokens 1280 \
                            --baseline-tokens 20100 --scope total    # MEASURED → 31.79%
python scripts/telemetry.py --output-file final.md --baseline-file draft.md   # ESTIMATED
python scripts/telemetry.py                                          # UNAVAILABLE
python scripts/telemetry.py --output-file final.md --query output    # one field
python scripts/telemetry.py ... --json                               # machine-readable
```

Savings tracking is opt-in because it costs tokens: Claude must write a full unoptimized draft to compare against.

## Installing

Open the `.skill` file card in Claude and choose **Save skill**, or upload the package in Claude's skills settings. Running `telemetry.py` inside Claude requires code execution to be enabled.

## Suggested test prompts

1. `efficient mode: boiling point of water at sea level in °F?` → one line, no council
2. `efficient mode: who is the current CEO of Netflix?` → searched, one line + source
3. `efficient mode: write a python function that dedupes a list, keeping order` → code only, no council
4. `quick one: SQLite or Postgres for a single-user desktop notes app?` → Tier 1 format
5. `run the council: is intermittent fasting good for longevity?` → ANALYZE, no workflow
6. `run the council: plan our 2 TB Postgres 13→16 migration, 4-hour window, one replica` → PLAN with critical path and per-step rollback
7. Follow-up to 6: `the window is now 2 hours` → only the changed steps
8. `show me the token dashboard` → UNAVAILABLE, no invented numbers
9. `what's 15% of 80, and should I refinance my mortgage at 6.1%?` → "12" now, plus one question about the current rate and term

## Limitations

- Estimates are a character-based heuristic, not tokenizer counts.
- Loop limits (one escalation, one P4 retest, two execution repairs, one re-verification) guarantee the process ends; when a limit is hit, Claude reports the state honestly with LOW confidence rather than looping.
- Council notes summarize decisions; they are not a transcript of Claude's private reasoning.

## Changelog

- **R5** — Evidence levels E1–E5 drive adjudication, confidence and verification; concrete thresholds for Tier 2 triggers; compound messages tiered per part; replies to clarifying questions resume the run; escalation can go 0 → 1 and jumps directly to the right tier; pre/post-compression gate merged with the self-check. SKILL.md shrank from ≈3,112 to ≈2,716 tokens (est., `telemetry.py`) while gaining these rules.
- **R4** — Restructured into this layout; added `telemetry.py`; savings scope (`output` | `total`) reconciles the original spec's 31.79% example; like-for-like comparability rule; acyclic dependency rule; critical-path check; per-task VERIFY/ROLLBACK; opt-in savings tracking.
- **R3** — Material triggers (no council for small code); ANALYZE/PLAN/ACT modes; Delta mode; deletion test; user controls; decision trail by default.
- **R2** — Logic fixes: P2 no-objection no longer skips P4–P7; adjudication rule; freshness at every tier; two-pass workflow; loop limits; consistent confidence criteria; defined quality gate.
- **R1** — Converted the AmelTech Chat Efficiency master instruction into a skill.
