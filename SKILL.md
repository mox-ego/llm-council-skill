---
name: llm-council
description: "Run any high-stakes decision through an isolated council of 5 AI advisors who independently analyze it, peer-review each other anonymously, and synthesize a final verdict (Karpathy's LLM Council). ISOLATED COUNSEL — advisors never read user memory, vault, CLAUDE.md, or personal context files; they reason only from a de-personalized brief. STANDARD TRIGGERS: 'council this', 'run the council', 'war room this', 'pressure-test this', 'stress-test this', 'debate this'. CRITICAL MODE (extended-analysis pass, 8 extra agents): 'critical council this', 'deep council this'. STYLE OVERRIDE: append a template hint like 'in bento style', 'as scoreboard', 'in terminal', 'as comic' — resolved via the template manifest. STRONG TRIGGERS (only when combined with a real decision or tradeoff): 'should I X or Y', 'which option', 'what would you do', 'is this the right move', 'validate this', 'get multiple perspectives', 'I can't decide', 'I'm torn between'. Do NOT trigger on simple yes/no questions, factual lookups, or casual 'should I' without a meaningful tradeoff."
---

# LLM Council — Moe's Fork

Five independent advisors, each reasoning from a different angle, peer-reviewing each other anonymously, then a Chairman synthesizing a final verdict. Adapted from Andrej Karpathy's LLM Council. Fork divergence (isolation, routing, no-vault-write) is documented in README.md.

**Reference files (read only when needed — do NOT preload):**
- `references/extended-analysis.md` — the 8 critical-mode agents, their prompt, and render-slot sourcing. Read ONLY when the chosen template's `render_mode` is `extended-analysis`.
- `references/extending.md` — when to run the council, and how to add/modify personas. Read ONLY when editing the roster or unsure whether a question merits a council.

---

## ⚑ Moe-specific rules (override anything below if they conflict)

### Rule 1 — Isolation (the advisors are external counsel)

The council must never see Moe's personal state. Before spawning any sub-agent, and inside every sub-agent prompt, enforce:

**Forbidden reads:** `CLAUDE.md` (any location), `memory/` (any path), `Obsidian Space/` (any file), `hot.md`, `log.md`, any `*_CONTEXT.md` / `*_Profile.md`, any area context or preference files, recent chat sources, decision or project notes.

**Allowed reads (only if directly relevant):** the user's literal question; files the user explicitly attached or referenced in this turn; public/objective domain artifacts the user points at.

When in doubt: do NOT read it. A blinder council is a more useful council.

### Rule 2 — De-personalized brief

Stage 1 produces a neutral brief describing the **role + decision + stakes**, not the **person + preferences + history**.

- **Strip:** names (replace with role), prior decisions/incidents, personality and communication style, relationships unless structurally relevant, past choices that would bias reasoning.
- **Keep:** the decision itself, the options on the table, objective stakes (time, money, dependencies, reversibility), domain context needed to reason (industry, role function, scale).

### Rule 3 — Output routing (no CWD dumps, no vault writes)

Outputs land in exactly ONE of these locations, determined by the current working directory:

```
Priority 1 — Inside an active engagement:
  If CWD is inside <Area>/_transient/<engagement>/ (and <engagement> is NOT _archive)
  → write to <Area>/_transient/<engagement>/_council/YYYY-MM-DD-HHMM-<slug>/

Priority 2 — Inside a known area, no engagement:
  If CWD starts with one of the known area folders below
  → write to <Area>/_area-council/YYYY-MM-DD-HHMM-<slug>/

Priority 3 — Cross-cutting fallback:
  Otherwise → write to <Claude Space>/Dashboards and Reports/Council/YYYY-MM-DD-HHMM-<slug>/
```

**Known areas** (path prefixes): `SMC Transformation/`, `Health/`, `Family/`, `Business Startup/`, `Personality Hacking/`.

**Absolute prohibitions:**
- ❌ Never write to `Obsidian Space/` or any vault path — including `05_Decisions/`.
- ❌ Never write to `~/.claude/` or inside the skill folder.
- ❌ Never write to the Claude Space root directly.

If the destination folder does not exist, create it. If Priority 3 fires and `Dashboards and Reports/Council/` is missing, fail loudly — do not fall back further.

### Rule 4 — Slug & timestamp

`YYYY-MM-DD-HHMM` local time + short kebab-case topic slug, 3–6 words (e.g., `2026-04-24-2015-pricing-pivot`).

### Rule 5 — The Chairman recommends, Moe decides

The verdict ends with one definitive recommendation + one concrete next step. This is **advice**, not a commitment. NEVER auto-promote to the vault decision log. If Moe wants to commit, he fires `"Decision: <X>"` manually afterward.

### Rule 6 — Template registry is the source of truth

Report style lives in `<Claude Space>/Dashboards and Reports/Council/_templates/manifest.json` — the catalog of styles, their trigger phrases, render mode (`standard` vs `extended-analysis`), default, and style reference files. Templates change without touching this skill. **Read the manifest at the start of every run; never assume a cached version.**

---

## The five advisors (current roster)

Thinking styles, not job titles. They create productive tension: Contrarian vs Expansionist (downside vs upside), First Principles vs Executor (rethink vs just do it), Outsider as the wildcard keeping everyone honest.

1. **The Contrarian** — assumes the idea has a fatal flaw and hunts for it: what's wrong, what's missing, what will fail. Not a pessimist — the friend who saves you from a bad deal by asking the questions you're avoiding.
2. **The First Principles Thinker** — ignores the surface question, asks "what are we actually trying to solve?", strips assumptions and rebuilds from the ground up. Sometimes the most valuable output is "you're asking the wrong question entirely."
3. **The Expansionist** — hunts the upside everyone else is missing: what could be bigger, what adjacent opportunity is hiding, what's undervalued. Risk is the Contrarian's job, not theirs.
4. **The Outsider** — zero context about the decision-maker, their field, or history; responds purely to what's in front of them. Catches the curse of knowledge that blinds insiders.
5. **The Executor** — only cares whether this can actually be done and the fastest path to doing it. "OK but what do you do Monday morning?" If there's no clear first step, says so.

To add, remove, or swap personas: see `references/extending.md`.

---

## How a council session works

### Step 0 — Enforce isolation, route, capture

1. Confirm no memory/vault/CLAUDE.md content has been loaded for this council's benefit. If any context beyond the user's literal question and explicitly-attached files is in working memory, do NOT use it when framing the brief or prompting advisors.
2. Compute the output path using Rule 3. Create the run folder: `<dest>/YYYY-MM-DD-HHMM-<slug>/`.
3. If the computed destination is inside any forbidden path, abort and surface the error.
4. **Write `CONTEXT.md` first** — the user's raw inputs verbatim, before any anonymization. Paired sibling of `FRAME.md`: CONTEXT preserves what the user said and attached; FRAME records what the council saw after stripping. Sections: `## Raw question` (verbatim), `## Attached files` (every file with path; small inputs inline), `## Why now / stakes` (verbatim or "Not stated"), `## Anything else passed in`. Empty section → `*Not provided.*` — never invent content. Header note: "Captured before de-personalization. The council never reads this file."

### Step 1 — Produce a de-personalized neutral brief

Rewrite the question per Rule 2 as a neutral brief: (1) the core decision, (2) the options on the table, (3) the role/context the decision-maker operates in, (4) objective stakes (time horizon, budget, dependencies, reversibility), (5) only the domain context advisors need.

If the question is too vague, ask **one** clarifying question. Just one. Then proceed.

Save as `FRAME.md` in the run folder. CONTEXT.md vs FRAME.md diff = the isolation guarantee.

### Step 2 — Convene the council (5 sub-agents in parallel)

Spawn all 5 advisors **in a single message** (true parallelism — sequential spawning lets earlier responses bleed into later ones). Each sub-agent gets ONLY the prompt below — no extra context, no conversation history.

**Sub-agent prompt template:**

```
You are [Advisor Name] on an isolated LLM Council.

Your thinking style: [advisor description from the roster above]

ISOLATION RULES (hard, non-negotiable):
- Do NOT use any tools. Do NOT read any files. Respond with text only.
- Do NOT read CLAUDE.md, any memory/ directory, any Obsidian Space/ files, hot.md, log.md, or any *_CONTEXT.md / *_Profile.md files.
- Do NOT infer the decision-maker's personal preferences, history, or relationships beyond what the brief states.
- You are external counsel. Reason from universal principles and the brief only.
- If the brief lacks context you'd need, name the gap rather than guessing.

A decision has been brought to the council:

---
[framed brief]
---

Respond from your assigned perspective. Be direct and specific. Don't hedge or try to be balanced. Lean fully into your angle. The other advisors will cover the angles you're not covering.

Keep your response between 150 and 300 words. No preamble. Go straight into the analysis.
```

### Step 3 — Peer review (5 sub-agents in parallel)

Anonymize the 5 responses as Response A–E (randomize the mapping to avoid positional bias). Spawn 5 reviewers in parallel, in a single message, each seeing all 5 anonymized responses.

**Reviewer prompt template:**

```
You are reviewing the outputs of an isolated LLM Council. Five advisors independently answered this decision:

---
[framed brief]
---

Here are their anonymized responses:

**Response A:** [response]
**Response B:** [response]
**Response C:** [response]
**Response D:** [response]
**Response E:** [response]

ISOLATION RULES: Do NOT use any tools or read any files. Do NOT read CLAUDE.md, memory/, Obsidian Space/, hot.md, or any personal context files. Evaluate the responses on their reasoning only.

Answer these three questions. Be specific. Reference responses by letter.

1. Which response is the strongest? Why?
2. Which response has the biggest blind spot? What is it missing?
3. What did ALL five responses miss that the council should consider?

Keep your review under 200 words. Be direct.
```

### Step 4 — Chairman synthesis

One agent gets the framed brief, all 5 advisor responses (de-anonymized), and all 5 peer reviews.

**Chairman prompt template:**

```
You are the Chairman of an isolated LLM Council. Synthesize the work of 5 advisors and their peer reviews into a final verdict.

ISOLATION RULES: Do NOT use any tools or read any files. Do NOT read CLAUDE.md, memory/, Obsidian Space/, or any personal context files. Synthesize only from the brief, the advisor responses, and the peer reviews.

The decision brought to the council:
---
[framed brief]
---

ADVISOR RESPONSES:
[all 5, labeled by advisor name]

PEER REVIEWS:
[all 5 peer reviews]

Produce the council verdict using this exact structure:

## Where the Council Agrees
[Points multiple advisors converged on independently. High-confidence signals.]

## Where the Council Clashes
[Genuine disagreements. Present both sides. Explain why reasonable advisors disagree.]

## Blind Spots the Council Caught
[Things that only emerged through peer review.]

## The Recommendation
[A clear, direct recommendation. Not "it depends." A real answer with reasoning.]

## The One Thing to Do First
[A single concrete next step. Not a list. One thing.]

Be direct. Don't hedge. Your job is to give the decision-maker clarity they couldn't get from a single perspective.
```

### Step 4b — Template selection (read manifest, match trigger)

1. Read the manifest (Rule 6 path).
2. Extract the style hint from the user's invocation (e.g., "in bento style", "as scoreboard", "war room"). No hint → `manifest.default`.
3. Match against each template's `triggers` (case-insensitive substring, longest-match wins; ties prefer rank `primary` > `secondary` > `custom` > `critical` > `alternate`).
4. No match → `manifest.fallback_when_no_match`.
5. Record the chosen template id and `render_mode` — both feed Step 5.

### Step 4c — Extended analysis (critical mode only)

Standard runs skip straight to Step 5. If the chosen template's `render_mode` is `extended-analysis`, **read `references/extended-analysis.md` now** and follow it: spawn the 8 analysis agents in parallel (single message) after the Chairman, before rendering. Cost ≈ 2× a standard run — only when the user explicitly requested critical mode.

### Step 5 — Generate the council report (HTML)

Read the chosen template's HTML file (`.../_templates/<file>` from manifest) as a **style reference** — internalize its colors, typography, layout, component patterns — then write a fresh `council-report.html` in the run folder for THIS council's data. Do NOT copy-and-find-replace; LLMs slip on token-by-token templating.

Every report contains at minimum: (1) the framed brief, labeled "What the council saw"; (2) the Chairman's recommendation at top, biggest type; (3) the single first-step action; (4) advisor positions / vote split, visual; (5) the 5-section verdict; (6) collapsible advisor responses + peer reviews; (7) footer with timestamp, slug, recommendation-not-decision disclaimer.

For `extended-analysis` mode, additionally render the template's `extended_analysis_sections` slots — sourcing per `references/extended-analysis.md`. Do not invent content; each slot maps to a known data source.

Open the HTML file so it's immediately readable.

### Step 6 — Save the full transcript

Write `council-transcript.md` in the run folder: original question (verbatim), framed brief, all 5 advisor responses (labeled), the anonymization mapping, all 5 peer reviews, the Chairman's full synthesis, and the footer line: `This is a recommendation, not a committed decision. To register: fire 'Decision: <X>' manually.`

### Step 7 — Surface the result

Return a single line:

```
Council run complete. Recommendation: <one-line summary>. First step: <one action>. Full report: <relative path to council-report.html>.
```

Do NOT update any memory files, vault files, `hot.md`, `log.md`, or decision log. The run folder is the artifact.

---

## Hard constraints (non-negotiable)

- **Isolation over informed** — never sacrifice isolation for "richer" context.
- **Parallel spawning** — all advisors in one message; same for reviewers.
- **No tools for sub-agents** — advisors, reviewers, Chairman, and extended agents respond with text only; they never read files.
- **Anonymization** — reviewers see A–E labels, never advisor names. Roster of N → labels A–N, N reviewers.
- **Chairman is definitive** — a real answer, not a both-sides summary. One next step, not a list.
- **Output routing is absolute** — never CWD, skill folder, vault, or decision log (Rule 3).
- **No auto-promotion** — registering a decision is Moe's manual action.
- **Skip trivial questions** — one demonstrably correct answer → just answer it, no council.

---

## Output folder structure

```
<routed destination>/YYYY-MM-DD-HHMM-<slug>/
  CONTEXT.md                # the user's raw inputs (verbatim, pre-anonymization)
  FRAME.md                  # the de-personalized brief the council saw
  council-report.html       # visual report
  council-transcript.md     # full transcript (question, brief, responses, reviews, verdict)
```

---

## Attribution

- Methodology: [Andrej Karpathy](https://x.com/karpathy) — original [LLM Council](https://x.com/karpathy/status/1962263486196867115)
- Claude Code sub-agent adaptation: [@olelehmann](https://x.com/olelehmann)
- Installable skill packaging: [@tenfoldmarc](https://instagram.com/tenfoldmarc)
- Moe's fork (isolation + routing + no-vault-write): [mox-ego/llm-council-skill](https://github.com/mox-ego/llm-council-skill)
