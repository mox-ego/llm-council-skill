---
name: llm-council
description: "Run any high-stakes decision through an isolated council of 5 AI advisors who independently analyze it, peer-review each other anonymously, and synthesize a final verdict. Based on Karpathy's LLM Council methodology. ISOLATED COUNSEL — advisors never read user memory, vault, CLAUDE.md, or personal context files; they reason only from a de-personalized brief. STANDARD TRIGGERS: 'council this', 'run the council', 'war room this', 'pressure-test this', 'stress-test this', 'debate this'. CRITICAL MODE TRIGGER (extended-analysis pass with 8 extra agents — risk asymmetry, reversibility ladder, info gap registry, time decay, pre-mortem, trip-wires, counterfactual, stakeholder impact): 'critical council this', 'deep council this'. STYLE OVERRIDE — append a template hint like 'in bento style', 'as scoreboard', 'in terminal', 'as comic'; the skill reads `Reports/Council/_templates/manifest.json` to map the hint to a template file. STRONG TRIGGERS (when combined with a real decision or tradeoff): 'should I X or Y', 'which option', 'what would you do', 'is this the right move', 'validate this', 'get multiple perspectives', 'I can't decide', 'I'm torn between'. Do NOT trigger on simple yes/no questions, factual lookups, or casual 'should I' without a meaningful tradeoff."
tier: hot
---

# LLM Council — Moe's Fork

Five independent advisors, each reasoning from a different angle, peer-reviewing each other anonymously, then a Chairman synthesizing a final verdict. Adapted from Andrej Karpathy's LLM Council. Claude Code adaptation inspired by the community.

This fork diverges from the upstream in three deliberate ways:

1. **Hard isolation** — advisors never read user memory, vault, CLAUDE.md, or personal context files. They see a de-personalized brief only. The council exists to challenge Moe's thinking, not to confirm his preferences.
2. **Output routing** — outputs land next to the work that prompted them (engagement → area → `Reports/`). Never dumped at the current working directory or inside the skill folder.
3. **No automatic decision-log promotion** — the Chairman's verdict is a recommendation, not a commitment. Promotion to the vault `05_Decisions/` log happens only when Moe manually fires `"Decision: X"` later.

---

## ⚑ Moe-specific rules (read before every run)

These override anything in the rest of this file if they conflict.

### Rule 1 — Isolation (the advisors are external counsel)

The council must never see Moe's personal state. Before spawning any sub-agent, and inside every sub-agent prompt, enforce:

**Forbidden reads:**
- `CLAUDE.md` (any location — root, area, vault)
- `memory/` directory (at any path)
- `Obsidian Space/` (any file)
- `hot.md`, `log.md`
- Any `*_CONTEXT.md`, `*_Profile.md` files
- Any area-specific context or preference files
- Recent chat sources, decision notes, or project notes

**Allowed reads (only if directly relevant to the question):**
- The user's literal question
- Files the user explicitly attached or referenced in this turn
- Public/objective domain artifacts (e.g., a landing page they're critiquing, a spec doc they wrote) — only if the user points at them

When in doubt: do NOT read it. A blinder council is a more useful council.

### Rule 2 — De-personalized brief

Stage 1 produces a neutral brief describing the **role + decision + stakes**, not the **person + preferences + history**. Strip:
- Names (replace with role: "a director of a project governance office")
- Prior decisions or incidents
- Personality traits, preferences, communication style
- Relationships unless structurally relevant
- Past choices that would bias the reasoning

Keep:
- The decision itself
- The options on the table
- Objective stakes (time, money, dependencies)
- Domain context the advisors need to reason (industry, role function, scale)

### Rule 3 — Output routing (no CWD dumps, no vault writes)

Outputs land in exactly ONE of these locations, determined by detecting the current working directory:

```
Priority 1 — Inside an active engagement:
  If CWD is inside <Area>/_transient/<engagement>/ (and <engagement> is NOT _archive)
  → write to <Area>/_transient/<engagement>/_council/YYYY-MM-DD-HHMM-<slug>/

Priority 2 — Inside a known area, no engagement:
  If CWD starts with one of the known area folders below
  → write to <Area>/_area-council/YYYY-MM-DD-HHMM-<slug>/

Priority 3 — Cross-cutting fallback:
  Otherwise (root of Claude Space, unknown folder, or outside any area)
  → write to <Claude Space>/Reports/Council/YYYY-MM-DD-HHMM-<slug>/
```

**Known areas** (match these as path prefixes):
- `SMC Transformation/`
- `Health/`
- `Family/`
- `Business/`
- `Personality Hacking/`
- `Hobbies/`
- `Leadership/`
- `Learning/`

**Absolute prohibitions:**
- ❌ Never write to `Obsidian Space/` or any vault path.
- ❌ Never write to `~/.claude/` or inside the skill folder.
- ❌ Never write to the Claude Space root directly.
- ❌ Never create or update any file under `Obsidian Space/05_Decisions/` — the Chairman's verdict is a recommendation, not a registered decision.

If the destination folder does not exist, create it. If Priority 3 fires and `Reports/Council/` is missing, fail loudly — do not fall back further.

### Rule 4 — Slug & timestamp

- **Timestamp**: `YYYY-MM-DD-HHMM` in local time.
- **Slug**: short kebab-case phrase capturing the decision topic (e.g., `2026-04-24-2015-pricing-pivot`). 3–6 words, lowercase, no punctuation except hyphens.

### Rule 5 — The Chairman recommends, Moe decides

The Chairman's verdict ends with a single definitive recommendation + one concrete next step. This is **advice**, not a decision Moe has committed to. The skill must NEVER auto-promote the verdict to the vault decision log. If Moe wants to commit, he'll fire `"Decision: <X>"` manually afterward.

### Rule 6 — Template registry is the source of truth

Report rendering style is NOT hardcoded in this file. It lives in:

```
<Claude Space>/Reports/Council/_templates/manifest.json
```

The manifest is the single source of truth for:
- Available report styles (the catalog)
- Trigger phrases that select each style
- Render mode per style (`standard` vs `extended-analysis`)
- The default style when no trigger is matched
- Style reference files (visual templates to mirror when generating output)

**Why this matters:** templates can be added, renamed, swapped, or visually retuned without touching this skill file. The skill's job is to (1) read the manifest, (2) match the user's invocation to a template, (3) render in that template's style. The catalog evolves; the skill stays put.

**Read the manifest at the start of every run.** Never assume a cached version. Templates change.

---

## When to run the council

The council is for questions where being wrong is expensive.

**Good council questions** (cross-domain — this fork is not marketing-specific):
- "Should I accept the EPMO secondment or stay on the Transformation track?"
- "Should I move my morning training to 5am or keep 6:30am and cut sleep elsewhere?"
- "Which of these three family-time redesigns keeps Fridays intact without hurting career velocity?"
- "Pitch the startup angle as Saudi-first or global-first?"
- "Replace the legacy auth middleware in one cutover or phase it over two quarters?"

**Bad council questions**:
- "What's the capital of France?" (one right answer)
- "Write me a tweet" (creation, not decision)
- "Summarize this article" (processing, not judgment)
- "Should I use markdown or rich text?" (trivial, no stakes)

The council tells you things you don't want to hear. That's the feature.

### Tier 0 — decision-helper first for low/medium stakes

Before convening the council, ask: is this decision **reversible within ~1 week** at low cost (a config tweak, a tooling pick, a draft route, a "try it Tuesday and switch back Friday" experiment)? If yes, fire `anthropic-skills:decision-helper` first. It's a single-agent structured-frame pass (criteria → options → recommendation) that costs a fraction of a full council run and is usually sufficient for tactical / reversible calls.

**Escalate from decision-helper to council** if any of the following surface during its run: (a) the recommendation is ambiguous or hedged, (b) the decision-helper itself flags high-stakes signals (irreversibility, cross-domain entanglement, stakeholder/legal/reputational risk), (c) the option space is wider than 3 and the criteria scoring is close, (d) Moe explicitly upgrades ("council this anyway").

Strategic / irreversible / multi-year / public / regulated decisions skip Tier 0 and go straight to council. Examples that bypass decision-helper: career moves, family commitments with multi-year reach, public commitments under the SMC mast, regulated-domain pivots, anything where the wrong call cannot be unwound in one work-week.

---

## The five advisors (current roster)

Each advisor thinks from a different angle. These are **thinking styles**, not job titles. They create productive tension with each other.

### 1. The Contrarian

Actively looks for what's wrong, what's missing, what will fail. Assumes the idea has a fatal flaw and tries to find it. If everything looks solid, digs deeper. The Contrarian is not a pessimist. They're the friend who saves you from a bad deal by asking the questions you're avoiding.

### 2. The First Principles Thinker

Ignores the surface-level question and asks "what are we actually trying to solve here?" Strips away assumptions. Rebuilds the problem from the ground up. Sometimes the most valuable council output is the First Principles Thinker saying "you're asking the wrong question entirely."

### 3. The Expansionist

Looks for upside everyone else is missing. What could be bigger? What adjacent opportunity is hiding? What's being undervalued? The Expansionist doesn't care about risk (that's the Contrarian's job). They care about what happens if this works even better than expected.

### 4. The Outsider

Has zero context about the decision-maker, their field, or their history. Responds purely to what's in front of them. This is the most underrated advisor. Experts develop blind spots. The Outsider catches the curse of knowledge: things that are obvious to an insider but confusing to everyone else.

### 5. The Executor

Only cares about one thing: can this actually be done, and what's the fastest path to doing it? Ignores theory, strategy, and big-picture thinking. The Executor looks at every idea through the lens of "OK but what do you do Monday morning?" If an idea sounds brilliant but has no clear first step, the Executor will say so.

**Why these five:** They create three natural tensions. Contrarian vs Expansionist (downside vs upside). First Principles vs Executor (rethink everything vs just do it). The Outsider sits in the middle keeping everyone honest by seeing what fresh eyes see.

See the **Adding or modifying personas** section at the bottom of this file for how to extend or swap the roster.

---

## How a council session works

### Step 0 — Enforce isolation, route, capture

Before doing anything else:

1. Confirm no memory/vault/CLAUDE.md content has been loaded for this council's benefit. If any context beyond the user's literal question and explicitly-attached files is in working memory, do NOT use it when framing the brief or when prompting advisors.
2. Compute the output path using Rule 3. Verify the parent folder exists (create if missing). Create the run folder: `<dest>/YYYY-MM-DD-HHMM-<slug>/`.
3. If the computed destination is inside any forbidden path (vault, skill folder, `~/.claude/`, `05_Decisions/`), abort and surface the error.
4. **Write `CONTEXT.md` first** — capture the user's raw inputs verbatim before any anonymization. This is the **paired sibling of `FRAME.md`**: CONTEXT preserves what the user actually said and attached; FRAME records what the council saw after stripping. Both live in the run folder so the user can return later and reconstruct exactly what was on the table. Use this template:

   ```markdown
   # Context — what the user gave the council

   > Captured before de-personalization. The council never reads this file.

   ## Raw question
   <verbatim — paste the user's question exactly as they wrote it>

   ## Attached files
   <list every file the user explicitly attached or pointed at, with its path.
    Copy small text inputs inline. For larger inputs, link to the file in the run folder.>

   ## Why now / stakes
   <verbatim if the user stated stakes; otherwise "Not stated">

   ## Anything else passed in
   <any other raw context the user supplied — links, screenshots, prior turns
    they pointed at. Verbatim.>
   ```

   If a section has nothing, write `*Not provided.*` — never invent content. The council still reads only `FRAME.md`; CONTEXT.md is for the user's own future reference.

### Step 1 — Produce a de-personalized neutral brief

Take the user's question and rewrite it as a neutral brief that:

1. States the core decision or question
2. Lists the options on the table (if any)
3. Describes the role/context the decision-maker is operating in (e.g., "director of a project governance office at a motorsport event organization; team of ~15; reports to C-suite")
4. States the objective stakes (time horizon, budget, dependencies, reversibility)
5. Adds only the domain context the advisors need to reason specifically

**Do NOT include:**
- The user's name or identifying details
- Their preferences, personality, or communication style
- Past decisions or incidents unless structurally relevant to THIS decision
- Anything read from `CLAUDE.md`, `memory/`, or the vault

If the question is too vague to produce a brief, ask **one** clarifying question. Just one. Then proceed.

Save the brief as `FRAME.md` in the run folder, alongside the `CONTEXT.md` written in Step 0.4. Both are auditable: `CONTEXT.md` shows what the user gave; `FRAME.md` shows what the council saw after stripping. The diff between them is the isolation guarantee.

### Step 2 — Convene the council (5 sub-agents in parallel)

> Model pins per efficient-fable roster (2026-06-12): advisors/chairman opus, reviewers sonnet — bare spawns inherit the session model at full rate.

Spawn all 5 advisors simultaneously as sub-agents in a single message with `model: "opus"`. Each gets:

1. Their advisor identity and thinking style (from the descriptions above)
2. The framed brief (from `FRAME.md`)
3. An explicit isolation instruction (see template)
4. An instruction to respond independently, directly, and without hedging

Each advisor produces a response of 150–300 words.

**Sub-agent prompt template:**

```
You are [Advisor Name] on an isolated LLM Council.

Your thinking style: [advisor description from above]

ISOLATION RULES (hard, non-negotiable):
- Do NOT read CLAUDE.md, any memory/ directory, any Obsidian Space/ files, hot.md, log.md, or any *_CONTEXT.md / *_Profile.md files.
- Do NOT infer the decision-maker's personal preferences, history, or relationships beyond what the brief states.
- You are external counsel. Reason from universal principles and the brief only.
- If the brief lacks context you'd need, name the gap in your response rather than guessing from other sources.

A decision has been brought to the council:

---
[framed brief]
---

Respond from your assigned perspective. Be direct and specific. Don't hedge or try to be balanced. Lean fully into your angle. The other advisors will cover the angles you're not covering.

Keep your response between 150 and 300 words. No preamble. Go straight into the analysis.
```

### Step 3 — Peer review (5 sub-agents in parallel)

Collect all 5 advisor responses. Anonymize them as Response A through E (randomize the mapping to avoid positional bias).

Spawn 5 new sub-agents in parallel with `model: "sonnet"`, one per reviewer. Each sees all 5 anonymized responses and answers three questions:

1. Which response is the strongest and why? (pick one)
2. Which response has the biggest blind spot and what is it?
3. What did ALL responses miss that the council should consider?

**Reviewer prompt template:**

```
You are reviewing the outputs of an isolated LLM Council. Five advisors independently answered this decision:

---
[framed brief]
---

Here are their anonymized responses:

**Response A:**
[response]

**Response B:**
[response]

**Response C:**
[response]

**Response D:**
[response]

**Response E:**
[response]

ISOLATION RULES: Do NOT read CLAUDE.md, memory/, Obsidian Space/, hot.md, or any personal context files. Evaluate the responses on their reasoning, not on any external knowledge about the decision-maker.

Answer these three questions. Be specific. Reference responses by letter.

1. Which response is the strongest? Why?
2. Which response has the biggest blind spot? What is it missing?
3. What did ALL five responses miss that the council should consider?

Keep your review under 200 words. Be direct.
```

### Step 4 — Chairman synthesis

One agent (`model: "opus"`) gets everything: the framed brief, all 5 advisor responses (now de-anonymized), and all 5 peer reviews. The Chairman produces the final council output using this exact structure:

**COUNCIL VERDICT**

1. **Where the council agrees** — the points multiple advisors converged on independently. High-confidence signals.
2. **Where the council clashes** — the genuine disagreements. Do not smooth these over. Present both sides and explain why reasonable advisors disagree.
3. **Blind spots the council caught** — things that only emerged through peer review. Things individual advisors missed that others flagged.
4. **The recommendation** — a clear, actionable recommendation. Not "it depends." Not "consider both sides." A real answer. The Chairman can disagree with the majority if the reasoning supports it.
5. **The one thing to do first** — a single concrete next step. Not a list. One thing.

**Chairman prompt template:**

```
You are the Chairman of an isolated LLM Council. Synthesize the work of 5 advisors and their peer reviews into a final verdict.

ISOLATION RULES: Do NOT read CLAUDE.md, memory/, Obsidian Space/, or any personal context files. Synthesize only from the brief, the advisor responses, and the peer reviews.

The decision brought to the council:
---
[framed brief]
---

ADVISOR RESPONSES:

**The Contrarian:**
[response]

**The First Principles Thinker:**
[response]

**The Expansionist:**
[response]

**The Outsider:**
[response]

**The Executor:**
[response]

PEER REVIEWS:
[all 5 peer reviews]

Produce the council verdict using this exact structure:

## Where the Council Agrees
[Points multiple advisors converged on independently. High-confidence signals.]

## Where the Council Clashes
[Genuine disagreements. Present both sides. Explain why reasonable advisors disagree.]

## Blind Spots the Council Caught
[Things that only emerged through peer review. Things individual advisors missed that others flagged.]

## The Recommendation
[A clear, direct recommendation. Not "it depends." A real answer with reasoning.]

## The One Thing to Do First
[A single concrete next step. Not a list. One thing.]

Be direct. Don't hedge. Your job is to give the decision-maker clarity they couldn't get from a single perspective.
```

### Step 4a.1 — Optional humanizer pass on the synthesis (EN, long briefs only)

After Chairman synthesis, run the synthesis text through `anthropic-skills:humanizer` if AND ONLY IF both conditions hold: (i) the rendered brief is **> 200 words**, AND (ii) the audience is **English**. Below 200 words, the humanizer's pattern-removal pass tends to over-edit and flatten signal; above 200 words, the synthesis often picks up AI-tells (rule-of-three rhythms, vague attributions, em-dash overuse) worth scrubbing.

**Hard isolation rule:** the humanizer pass runs ONLY on the Chairman's final synthesis text. Advisors' raw responses and peer reviews stay isolated and un-humanized — their reasoning is the input record and must not be rewritten. The humanizer is a finishing-pass on the surfaced verdict only.

Arabic synthesis is out of scope for the humanizer (it's English-tuned). For Arabic deliverables of council verdicts, route through `arabic-style` per its tier protocol.

### Step 4b — Template selection (read manifest, match trigger)

Before rendering, choose which template to render to.

1. **Read the manifest:** `<Claude Space>/Reports/Council/_templates/manifest.json`.
2. **Extract the trigger phrase from the user's invocation.** Look for style hints in the original question (after "council this," before the colon, or in inline phrases like "in bento style" / "as scoreboard" / "war room"). If no style hint is present, use `manifest.default`.
3. **Match the trigger.** For each template in `manifest.templates`, check if any of its `triggers` substrings appear in the user's invocation (case-insensitive, longest-match wins). If multiple templates tie, prefer rank order: `primary` > `secondary` > `custom` > `critical` > `alternate`.
4. **If no match,** use `manifest.fallback_when_no_match`.
5. **Record the chosen template id and render mode.** Both feed Step 5.

### Step 4c — Extended analysis (only if `render_mode === "extended-analysis"`)

Standard runs go straight to Step 5. Extended runs do this first.

When the chosen template's `render_mode` is `extended-analysis`, spawn additional analysis sub-agents in parallel **after** Step 4 (Chairman synthesis) and **before** Step 5 (rendering). Each agent observes the same isolation rules and gets the framed brief + Chairman's verdict as input.

**The 8 extended-analysis agents** (parallel, single message, all spawned at once; use `model: "sonnet"` for each):

| # | Agent | Output goes to template slot | Output spec |
|---|---|---|---|
| 1 | `risk_asymmetry` | Risk Asymmetry | If-wrong cost vs if-right benefit, magnitude estimate per side, 1-line description per side. |
| 2 | `reversibility_ladder` | Reversibility Ladder | 4–7 sequential steps from "fully recoverable" → "sunk cost," each with a recoverability marker. |
| 3 | `info_gap_registry` | Information Gap Registry | 4–8 unknowns weighted HIGH/MED/LOW by answer-flipping power, with source attribution and how-to-close. |
| 4 | `time_decay_curve` | Time-Decay Curve | When does delay flip from helpful to harmful? Output a peak point and a flip point with rationale. |
| 5 | `pre_mortem` | Pre-Mortem | 3 most-likely failure modes branched by recommendation path (chosen / opposite / risky variant). |
| 6 | `trip_wires` | Trip-Wires | 4–6 dynamic monitor signals over weeks/months that would prompt reconsideration. |
| 7 | `counterfactual` | Counterfactual | 3–5 conditions that would flip the recommendation (with direction: "→ buy" / "→ wait" / "→ don't"). |
| 8 | `stakeholder_impact` | Stakeholder Impact Map | 4–8 affected parties with direction (+/−) and magnitude (HIGH/MED/LOW). |

**Sub-agent prompt template (extended-analysis agent):**

```
You are the [Agent Name] sub-agent in an isolated LLM Council, extended-analysis mode.

ISOLATION RULES (hard, non-negotiable):
- Do NOT read CLAUDE.md, memory/, Obsidian Space/, hot.md, or any personal context files.
- Reason only from the brief and the Chairman's verdict provided below.
- Do NOT use any other tools. Just respond.

THE BRIEF:
---
[framed brief]
---

THE CHAIRMAN'S VERDICT:
---
[chairman output]
---

YOUR TASK: [agent-specific output spec from the table above]

Output format: [structured spec — JSON or markdown depending on slot. See render template for layout].

Keep your output tight. No preamble. No fluff. Just the structured analysis.
```

**Important:** Extended-analysis agents do NOT need to be domain-specialized. The `pre_mortem` agent is the same agent class as the `trip_wires` agent — only the prompt task differs. They're 9 generic Claude sub-agents with 9 different task strings.

**Cost:** roughly 2× the standard run (5 advisors + 5 reviewers + 1 chairman + 8 analysts + 1 render = 20 calls). Only invoke when the user explicitly requests critical mode.

### Step 5 — Generate the council report (HTML)

**Read the chosen template's HTML file** (path = `<Claude Space>/Reports/Council/_templates/<file>` from manifest) to mirror its visual style — colors, typography, layout, component patterns. Then write a fresh `council-report.html` inside the run folder using that style.

**Do NOT copy the template verbatim and find-replace.** LLMs slip on token-by-token templating. Instead: read the template as a *style reference*, internalize its visual grammar, then re-render for THIS council's data.

Every report contains (at minimum):
1. The framed brief (clearly labeled "What the council saw")
2. The Chairman's recommendation prominently displayed (top of page, biggest type)
3. The single first-step action
4. Advisor positions / vote split (visual)
5. The 5-section verdict (agreement, clash, blind spots, recommendation, first step)
6. Collapsible advisor responses + peer review
7. Footer: timestamp, slug, recommendation-not-decision disclaimer

**For `extended-analysis` mode, additionally render** all the slots in the chosen template's `extended_analysis_sections` (manifest field). Each slot is sourced one of two ways:

- **Agent-produced** (8 slots): `risk_asymmetry`, `reversibility_ladder`, `info_gap_registry`, `time_decay_curve`, `pre_mortem`, `trip_wires`, `counterfactual`, `stakeholder_impact` → render the corresponding agent's verbatim output.
- **Derived from standard pipeline** (4 slots): `kpi_strip` (computed: confidence = peer-review unanimity, vote split = camp tally, regret asymmetry = compare risk_asymmetry magnitudes, reversibility cost = top of reversibility_ladder, etc.), `agree_clash` (extracted from Chairman's verdict sections), `council_positions` (each advisor's stance + lens, already known from Stage 2), `decision_journal` (template stub + recommendation + confidence; final review fields stay blank for the user to fill at outcome review).

Do not invent content. Each slot maps to a known data source.

Open the HTML file so it's immediately readable.

### Step 6 — Save the full transcript

Write `council-transcript.md` in the run folder. Contains:

- The original user question (verbatim)
- The framed brief (from FRAME.md)
- All 5 advisor responses (labeled by advisor)
- The anonymization mapping (which advisor was Response A, B, etc.)
- All 5 peer reviews
- The Chairman's full synthesis
- A footer line: `This is a recommendation, not a committed decision. To register: fire 'Decision: <X>' manually.`

### Step 7 — Surface the result

Return to the user a single-line summary:

```
Council run complete. Recommendation: <one-line summary>. First step: <one action>. Full report: <relative path to council-report.html>.
```

Do NOT update any memory files, vault files, `hot.md`, `log.md`, or decision log. The run folder is the artifact.

---

## Hard constraints (non-negotiable)

- **Isolation over informed**: never sacrifice isolation for "richer" context. A blinder council is more useful than a biased one.
- **Parallel spawning**: all 5 advisors must spawn in a single message (true parallelism). Sequential spawning defeats peer review by letting earlier responses bleed into later ones.
- **Anonymization**: peer reviewers see A–E labels, never advisor names. If the roster grows to N advisors, anonymization labels extend to A–N and Step 3 spawns N reviewers.
- **Chairman is definitive**: recommendation must be a real answer, not a both-sides summary. One concrete next step, not a list.
- **Output routing is absolute**: never write to CWD, skill folder, vault, or decision log. Always follow Rule 3's priority order.
- **No auto-promotion**: never register the verdict as a decision anywhere. Promotion is Moe's manual action.
- **Skip trivial questions**: if the user's input has one demonstrably correct answer, just answer it directly without running the council.

---

## Output folder structure

Every council run produces:

```
<routed destination>/YYYY-MM-DD-HHMM-<slug>/
  CONTEXT.md                # the user's raw inputs (verbatim, pre-anonymization)
  FRAME.md                  # the de-personalized brief the council saw
  council-report.html       # visual report
  council-transcript.md     # full transcript (question, brief, responses, reviews, verdict)
```

`CONTEXT.md` and `FRAME.md` are paired siblings. CONTEXT preserves what the user actually said + attached; FRAME records what the council saw after stripping. The user can return to the folder months later and reconstruct exactly what was on the table. Inspect the HTML for the verdict. Open the transcript when you need to dig into a specific advisor's reasoning.

---

## Adding or modifying personas

The skill is Moe's fork — he owns it. To add, remove, or swap personas:

1. **Edit this SKILL.md in three places:**
   - **"The five advisors" section** — add/remove/edit the persona description (keep it to 1 short paragraph: what they optimize for, what tensions they create).
   - **Step 2 spawn count** — if the count changes (e.g., 5 → 7), update the prose to match. The prompt template itself is persona-agnostic and needs no change.
   - **Step 3 anonymization labels** — extend from A–E to A–<N-th letter>, and update the prose to "Spawn N reviewers, one per advisor."

2. **Keep the tension principle alive.** The 5-advisor roster works because it manufactures three natural tensions (Contrarian vs Expansionist, First Principles vs Executor, Outsider as wildcard). If you add a persona, make sure it creates a NEW tension or sharpens an existing one — don't add a redundant angle.

3. **Commit to the fork** with a clear message like `add persona: The Long-Termer` or `replace Outsider with Domain Expert for Health decisions`.

4. **Future upgrade (not day-one):** a library of 15+ personas, with the skill dynamically picking 5–7 based on detected domain. Worth it once you've run ~20 councils and feel the fixed roster repeating itself. Seed it as a plant-seed when the pattern appears.

**Test your change** with a low-stakes question before running it on something real. If a persona produces generic or repetitive output in testing, tune the description or remove it.

---

## Personas

The standard 5-advisor roster (Contrarian / First Principles / Expansionist / Outsider / Executor) above is the council's general-purpose lineup — domain-agnostic, optimised for tension manufacturing.

**Domain-specific persona libraries** can be loaded by other skills WITHOUT modifying this council. The first such library shipped 2026-05-10 for SMC commercial policy work:

- **`policy-personas` skill** — 5 SME critique lenses (Saudi Legal Counsel, IFRS-15 External Auditor, Commercial Director, Internal Auditor, Customer/Corporate Sponsor). Lives at `~/.claude/skills/policy-personas/personas/`. Loaded by `/policy-builder` (Step 4) and `/policy-reviewer` (Step 4) — orthogonal to this council, not a replacement.

When other domain libraries appear (motorsport sporting, ops, marketing/CSR), they follow the same pattern: NEW additive skill at `~/.claude/skills/<domain>-personas/personas/`, loaded by domain orchestrators. This council remains untouched as the cross-domain tension-manufacturer.

If you ever want to run THIS council's standard 5-advisor protocol against a domain-specific brief, that's a parallel pass — not a substitution.

---

## Attribution

- Methodology: [Andrej Karpathy](https://x.com/karpathy) — original [LLM Council](https://x.com/karpathy/status/1962263486196867115)
- Claude Code sub-agent adaptation: [@olelehmann](https://x.com/olelehmann)
- Installable skill packaging: [@tenfoldmarc](https://instagram.com/tenfoldmarc)
- Moe's fork (isolation + routing + no-vault-write): [mox-ego/llm-council-skill](https://github.com/mox-ego/llm-council-skill)
