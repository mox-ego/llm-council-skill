# Before / after — same council, two design eras

Both files render the **same fixture council run** (the auth-middleware cutover-vs-phased
decision from SKILL.md's example list — no personal data) so the only variable is the
design system.

| File | Era | What it shows |
|---|---|---|
| `before-upstream-style.html` | Initial publish (commit `0dc0327`) | The original Step 5 spec: white background, system fonts, subtle borders, soft pastel accents per advisor, "nothing flashy — looks like a professional briefing document." Verdict is body text; positions are a plain table. |
| `after-design-system.html` | Current (`design-system.md`) | Token-driven render on the war-room skin: command-bar header, display-type verdict hero with gradient highlight + glowing confidence chip, FIRST STEP action bar, scoreboard-grade KPI strip, grouped vote-split bar with striped DISSENT segment, advisor cards with identity glows and monograms, ghost-numbered verdict sections, dossier-style FRAME.md panel, native `<details>` transcript with word counts, terminal-prompt footer. Layered glow/grid atmosphere, print-safe, 360px-responsive, zero external requests. |

These are **style references for humans**, not runtime templates — the runtime catalog
stays in `Dashboards and Reports/Council/_templates/manifest.json` per Rule 6. The design
grammar both eras are measured against is `../design-system.md`.
