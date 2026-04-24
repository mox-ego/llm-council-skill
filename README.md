# LLM Council — Moe's Fork

Fork of [tenfoldmarc/llm-council-skill](https://github.com/tenfoldmarc/llm-council-skill), customized for my Claude Space setup.

Based on [Andrej Karpathy's LLM Council](https://x.com/karpathy/status/1962263486196867115) methodology — five advisors debate, peer-review each other anonymously, and a Chairman synthesizes a verdict.

---

## What diverges from upstream

1. **Hard isolation from personal context.**
   Advisors never read `CLAUDE.md`, `memory/`, `Obsidian Space/`, `hot.md`, or any personal preference files. They reason from a de-personalized brief describing *role + decision + stakes*, not *person + preferences + history*. Upstream scans the workspace and leaks personal context into the framing — this fork explicitly forbids that.

2. **Output routing, not CWD dumps.**
   Outputs land inside a project's `_transient/` folder when the council runs during an engagement; inside the area's `_transient/_council/` when no engagement is active; and inside `Dashboards and Reports/Council/` as a cross-cutting fallback. Never at CWD, never in the skill folder, never in the vault.

3. **No automatic decision-log promotion.**
   The Chairman's verdict is a recommendation, not a committed decision. The skill NEVER writes to `Obsidian Space/05_Decisions/`. If I want to commit to a recommendation, I fire `"Decision: X"` manually afterward — that's the firewall.

4. **Cross-domain by design.**
   Upstream README leans marketing/business examples. This fork's examples and framing explicitly cover SMC governance, Health, Family, Business Startup, and Personality Hacking decisions. The 5 personas are generic cognitive lenses, not business-specific.

5. **Run-folder structure.**
   Each run produces `FRAME.md` + `council-report.html` + `council-transcript.md` inside `YYYY-MM-DD-HHMM-<slug>/`. `FRAME.md` is auditable — I can verify what the council actually saw.

---

## Install

Clone this fork into the skills directory:

```bash
git clone https://github.com/mox-ego/llm-council-skill \
  "C:/Users/moeal/Claude Space/.claude/skills/llm-council"
```

Claude Code auto-loads it.

---

## Use

Any of these triggers followed by the decision:

- `council this`
- `run the council`
- `pressure-test this`
- `stress-test this`
- `war room this`
- `debate this`

Also fires on decision-shaped strong triggers like `"should I X or Y"`, `"which option"`, `"validate this"` when combined with a real tradeoff.

**Example:**

> council this: should I move EPMO training delivery fully in-house or keep the current vendor partnership for another cycle?

Give it the options, the stakes, the timeline. The de-personalizing happens in Stage 1 — you can write the question naturally.

---

## Output locations

| Where you are | Output lands in |
|---|---|
| Inside `<Area>/_transient/<engagement>/` | `<Area>/_transient/<engagement>/_council/YYYY-MM-DD-HHMM-<slug>/` |
| Inside `<Area>/` (no engagement) | `<Area>/_transient/_council/YYYY-MM-DD-HHMM-<slug>/` |
| Anywhere else | `Dashboards and Reports/Council/YYYY-MM-DD-HHMM-<slug>/` |

Engagement-scoped runs archive automatically with the engagement. `Dashboards and Reports/Council/` runs need manual triage (see `Dashboards and Reports/README.md`).

---

## Adding / modifying personas

See the **Adding or modifying personas** section at the end of `SKILL.md`. Short version: edit SKILL.md in three spots (persona description, spawn count, anonymization labels), commit to the fork, test on a low-stakes question.

---

## License

MIT (inherited from upstream).

---

*Forked 2026-04-24 from [tenfoldmarc/llm-council-skill](https://github.com/tenfoldmarc/llm-council-skill).*
