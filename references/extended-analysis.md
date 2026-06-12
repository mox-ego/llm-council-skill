# Extended analysis (critical mode)

Read this file ONLY when the chosen template's `render_mode` is `extended-analysis`. It defines Step 4c (the 8 analysis agents) and the extended render-slot sourcing for Step 5.

## Step 4c — The 8 extended-analysis agents

Spawn all 8 in parallel, in a single message, **after** Step 4 (Chairman synthesis) and **before** Step 5 (rendering). Each observes the same isolation rules and receives the framed brief + the Chairman's verdict as input.

| # | Agent | Template slot | Output spec |
|---|---|---|---|
| 1 | `risk_asymmetry` | Risk Asymmetry | If-wrong cost vs if-right benefit, magnitude estimate per side, 1-line description per side. |
| 2 | `reversibility_ladder` | Reversibility Ladder | 4–7 sequential steps from "fully recoverable" → "sunk cost," each with a recoverability marker. |
| 3 | `info_gap_registry` | Information Gap Registry | 4–8 unknowns weighted HIGH/MED/LOW by answer-flipping power, with source attribution and how-to-close. |
| 4 | `time_decay_curve` | Time-Decay Curve | When does delay flip from helpful to harmful? A peak point and a flip point with rationale. |
| 5 | `pre_mortem` | Pre-Mortem | 3 most-likely failure modes branched by recommendation path (chosen / opposite / risky variant). |
| 6 | `trip_wires` | Trip-Wires | 4–6 dynamic monitor signals over weeks/months that would prompt reconsideration. |
| 7 | `counterfactual` | Counterfactual | 3–5 conditions that would flip the recommendation (with direction: "→ buy" / "→ wait" / "→ don't"). |
| 8 | `stakeholder_impact` | Stakeholder Impact Map | 4–8 affected parties with direction (+/−) and magnitude (HIGH/MED/LOW). |

The agents are NOT domain-specialized — 8 generic sub-agents with 8 different task strings.

**Cost:** roughly 2× the standard run (5 advisors + 5 reviewers + 1 chairman + 8 analysts + 1 render = 20 calls). Only invoke when the user explicitly requested critical mode.

## Sub-agent prompt template

```
You are the [Agent Name] sub-agent in an isolated LLM Council, extended-analysis mode.

ISOLATION RULES (hard, non-negotiable):
- Do NOT use any tools. Do NOT read any files. Respond with text only.
- Do NOT read CLAUDE.md, memory/, Obsidian Space/, hot.md, or any personal context files.
- Reason only from the brief and the Chairman's verdict provided below.

THE BRIEF:
---
[framed brief]
---

THE CHAIRMAN'S VERDICT:
---
[chairman output]
---

YOUR TASK: [agent-specific output spec from the table above]

Output format: [structured spec — JSON or markdown depending on slot. See render template for layout.]

Keep your output tight. No preamble. No fluff. Just the structured analysis.
```

## Step 5 — Extended render-slot sourcing

Render every slot in the chosen template's `extended_analysis_sections` (manifest field). Each slot is sourced one of two ways:

- **Agent-produced (8 slots):** `risk_asymmetry`, `reversibility_ladder`, `info_gap_registry`, `time_decay_curve`, `pre_mortem`, `trip_wires`, `counterfactual`, `stakeholder_impact` → render the corresponding agent's verbatim output.
- **Derived from the standard pipeline (4 slots):**
  - `kpi_strip` — computed: confidence = peer-review unanimity; vote split = camp tally; regret asymmetry = compare `risk_asymmetry` magnitudes; reversibility cost = top of `reversibility_ladder`.
  - `agree_clash` — extracted from the Chairman's verdict sections.
  - `council_positions` — each advisor's stance + lens (already known from Step 2).
  - `decision_journal` — template stub + recommendation + confidence; final review fields stay blank for the user to fill at outcome review.

Do not invent content. Each slot maps to a known data source.
