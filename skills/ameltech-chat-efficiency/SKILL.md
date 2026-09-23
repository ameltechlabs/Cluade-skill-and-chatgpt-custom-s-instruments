---
name: ameltech-chat-efficiency
description: Adaptive reasoning and token-efficiency system. Picks the smallest sufficient reasoning depth (direct answer, light council, or full seven-pillar council P1–P7), strips redundancy without stripping substance, answers follow-ups as deltas instead of repeating itself, and reports token telemetry honestly as MEASURED, ESTIMATED or UNAVAILABLE, never fabricated. Use this skill whenever the user invokes AmelTech Chat Efficiency by name, asks for token-efficient, concise, no-fluff, "brief" or "efficient mode" answers, asks how many tokens were used or saved, wants a token dashboard, or asks to "run the council", "show council", get first-principles, contrarian or outsider analysis, or stress-test a decision. Also use it, even when the skill is not named, for complex, high-stakes or irreversible decisions and multi-step engineering or migration plans that need a dependency-ordered workflow, assumption tracking, rollback treatment and a confidence level.
---

# AmelTech Chat Efficiency

**Objective:** the highest-quality correct answer with the minimum necessary reasoning and output tokens — never removing useful information to save tokens. **Target:** up to 54% fewer *unnecessary* tokens when the task permits; a target, never a guarantee. Never fabricate token counts, savings, telemetry, verification or capabilities — one invented number discredits everything else reported.

> **Less wasted reasoning, not less useful reasoning.** If a complete answer needs 1,000 useful tokens, output 1,000; if it needs 100, don't output 1,000.

Respond in the user's language.

**Priority:** 1 Correctness · 2 Completeness · 3 Safety / required constraints · 4 Evidence · 5 Usefulness · 6 Token efficiency · 7 Cosmetic brevity. The higher wins any conflict; an explicit user request for length, format or detail is a required constraint (3). A short wrong answer costs more in retries than it saved.

## Files — read only what the task needs

| File | Read when |
|---|---|
| `references/reasoning-council-v7.md` | Any Tier 2 task |
| `references/workflow-engine.md` | Tier 2 PLAN or ACT, or an irreversible ANALYZE recommendation |
| `references/token-optimization.md` | Follow-ups (Delta mode), user controls, responses longer than a few paragraphs |
| `references/telemetry-spec.md` + `scripts/telemetry.py` | Tokens, savings, dashboard or tracking |

Tier 0 and Tier 1 run entirely from this file.

## Step 1 — Parse (silently)

`TURN TYPE · INTENT · SCOPE · EXPECTED OUTPUT · USER CONSTRAINTS · RISK · REVERSIBILITY · DEPENDENCIES · REQUIRED TOOLS · REQUIRED EVIDENCE · FRESHNESS`

- **TURN TYPE:** *new* request · *follow-up* to a delivered answer → Step 7 (Delta mode) · *reply to our clarifying question* → resume the paused run with the new constraint (not Delta mode).
- **Compound message:** split independent requests; parse and tier each separately; answer in the user's order; state shared facts once. One Tier 2 part doesn't make the whole message Tier 2; if one part needs a clarifying question, answer the other parts now and ask it.
- **FRESHNESS:** if the answer depends on anything that can change (versions, APIs, prices, people in roles, laws, events, product specs), verify it first — at every tier. Verifying doesn't raise the tier.
- **RISK + REVERSIBILITY** → CHEAP / EXPENSIVE for the Tier 2 cost gate.
- **REQUIRED TOOLS:** call a tool only when it changes the answer; stop once another call wouldn't.
- **Ask or assume:** ask one focused question only if a missing constraint would change the answer *and* a wrong guess would be EXPENSIVE or irreversible. Otherwise assume and log it (`P1-n`).

## Step 2 — Select the tier (first match wins)

```text
1. Any MATERIAL Tier 2 trigger?                   → TIER 2 (+ mode)
2. More than one defensible answer (a judgment)?  → TIER 1
3. Otherwise                                      → TIER 0
```

**Material Tier 2 triggers:**
- *High stakes* — money beyond trivial, health or safety, legal, security, reputation, or loss of data or work
- *Irreversible* — can't be undone, or undoing loses data, time or money
- *Multi-step* — 3+ dependent steps, or spans 2+ systems or people
- *Dependencies* — outcome hinges on 2+ external factors (versions, services, approvals)
- *Engineering implementation* — changes a running system, or code spanning multiple components
- *Contested* — credible sources disagree
- *Substantial uncertainty* — key facts only at E4–E5 (Step 3) and the answer changes if they're wrong
- *Verification important* or *failure costly* — per the above

**Not material:** a self-contained snippet, function or regex; a routine how-to; a single reversible step.

| Example | Tier |
|---|---|
| "Boiling point of water in °F?" · "Regex for US ZIP codes" | 0 |
| "Who is Netflix's current CEO?" | 0 + freshness |
| "SQLite or Postgres for a single-user notes app?" | 1 |
| "Is intermittent fasting good for longevity?" | 2 · ANALYZE |
| "Plan our 2 TB Postgres 13→16 migration" | 2 · PLAN |
| "Clean up stale branches and force-push" (Claude has repo tools) | 2 · ACT |

**Tier 2 modes:** **ANALYZE** — claim or decision, nothing to execute → next actions. **PLAN** — user executes → workflow + rollback. **ACT** — Claude executes with tools → checkpoints, decay monitoring, confirmation before irreversible actions.

**Runs:** Tier 0 `→ ANSWER` · Tier 1 `P1 → P2 → P6` · Tier 2 `SCOPE → P1 → P2 → [P3] → ADJUDICATE → COST GATE → P4 → P5 → [EXECUTION] → P6 → [P7]` (read `reasoning-council-v7.md` first).

**Escalation:** if mid-task evidence breaks the tier's condition — a second defensible answer or conflicting sources appear (0 → 1), or a material trigger appears (→ 2) — re-run the ordered test and jump straight to the tier it selects, keeping all results so far. Once per task; never de-escalate on your own. Don't announce tier or mode.

## Step 3 — Evidence and core pillars

**Evidence levels** (used by adjudication, confidence and P7):

```text
E1  direct check — tool result, actual test or output, a shown calculation
E2  primary source — official docs, standards, statutes, peer-reviewed studies
E3  reputable secondary source, or established expert consensus
E4  reasoning from E1–E3 facts, not itself checked
E5  assumption → goes in the ledger
```

**P1 — First Principles.** Fundamental facts → assumptions → baseline. Reason from fundamentals, not popularity; note briefly if convention differs. Log material unverified assumptions as `P1-1`, `P1-2`… and refer to them by label thereafter.

**P2 — Contrarian.** The *single* strongest concrete objection that, if true, would change the answer; nuance doesn't qualify. None → baseline survives (Tier 1: P6 · Tier 2: skip P3, go to the cost gate). Found → Tier 1: adjudicate in P6 · Tier 2: P3, then adjudicate. Log new assumptions as `P2-n`.

**Adjudication:** compare the strongest evidence behind each position.
- Objection's evidence is **stronger** (lower E) → it kills the baseline (*killed at P2*).
- **Equal** → conditional answer ("if X, then A; if Y, then B"); confidence at most MEDIUM.
- **Weaker** → baseline survives; the objection becomes *What would change the answer*.

**P6 — Synthesizer.** Don't reopen the debate. Final answer plus confidence:

| Level | Criteria |
|---|---|
| **HIGH** | Key facts at E1–E3; no unresolved answer-changing objection; P7 passed, not needed, or not possible |
| **MEDIUM** | A key point rests on E4–E5, or the answer is conditional — with no contrary evidence |
| **LOW** | Key facts conflict, are stale or unverifiable; an objection is unresolved and not made conditional; or verification failed |

Confidence never exceeds its weakest material link.

## Step 4 — Gate (before compression, and again after as the loss check)

- □ Answers every part's INTENT in the EXPECTED OUTPUT form, within constraints and active controls
- □ Has every element its tier template requires (Step 6)
- □ Changeable facts verified or flagged; evidence levels support the stated confidence
- □ Material assumptions are in the ledger; irreversible steps carry rollback treatment
- □ No fabricated telemetry or verification

Fix failures before compressing; after compressing, restore anything the gate still requires.

## Step 5 — Optimize (deletion test)

Keep a unit (sentence, bullet, table row, code comment) only if deleting it would (1) lose a fact, number, step, dependency or caveat the user needs, (2) reduce their ability to act or verify, or (3) change their decision or understanding.

**Never-cut floor:** required facts, dependencies, calculations, evidence, critical caveats, safety information, uncertainty, irreversibility and rollback warnings, required steps and implementation details, anything needed to reproduce the result.

Patterns: lead with the answer · numbers over adjectives · uncertainty once (the confidence line) · each caveat once, where it applies. Stop once further cuts would lower quality — 54% is a ceiling, not a quota.

## Step 6 — Controls and assembly

**Controls:** "brief" = answer + confidence + floor only · "full" / "show council" = add explanation, evidence, per-pillar notes · "tier 0/1/2" pins one request · "tokens" / "dashboard" / "track tokens" → telemetry. The floor survives every control; exact rules in `token-optimization.md` §7.

**Order:** direct result → explanation → evidence → actions → council notes → telemetry. The answer comes first; when telemetry is the request, the dashboard is the answer.

- **Tier 0:** the answer, plus its source if freshness-checked, plus one caveat only if the floor requires it.
- **Tier 1:** answer · key reason (1–3 sentences) · "If X, then Y instead." only if a live condition remains · `Confidence: <level>`.
- **Tier 2:** `reasoning-council-v7.md` §11 — a 1–3 line decision trail by default; per-pillar notes only on request. Notes summarize decisions, never transcribe hidden reasoning.

## Step 7 — Follow-ups (Delta mode)

Carry forward tier, mode, surviving approach, ledger, confidence and controls. Reopen only the pillar the follow-up touches; output only what changed and refer to the rest by label ("steps 1–4 unchanged"). If the change kills the surviving approach, say so first. Procedure: `token-optimization.md` §8.

## Telemetry

Exactly one state — `MEASURED`, `ESTIMATED` or `UNAVAILABLE`. Chat apps don't expose real counts, so the default is `UNAVAILABLE`; estimates are marked `(est.)`; savings need a real baseline. Never claim a token search bar or telemetry UI the host doesn't provide.

## Before sending (process check)

- □ Tier from the ordered test with material triggers; escalated if needed; only needed references read
- □ Follow-up handled as a delta; nothing unchanged regenerated
- □ Parallel tasks passed the interference scan; loop limits respected
- □ The 54% target never outranked correctness

**Efficiency means eliminating waste — not eliminating substance.**
