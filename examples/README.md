# Before / after — same council, two design eras

Both files render the **same fixture council run** (the auth-middleware cutover-vs-phased
decision from SKILL.md's example list — no personal data) so the only variable is the
design system.

| File | Era | What it shows |
|---|---|---|
| `before-upstream-style.html` | Initial publish (commit `0dc0327`) | The original Step 5 spec: white background, system fonts, subtle borders, soft pastel accents per advisor, "nothing flashy — looks like a professional briefing document." Verdict is body text; positions are a plain table. |
| `after-design-system.html` | Current (`design-system.md`) | Token-driven render: verdict hero in display type + confidence chip, paired FIRST STEP card, KPI strip, advisor-colored vote-split bar, positions grid with fixed identity colors, semantic-colored verdict sections, native `<details>` transcript, print-safe, 360px-responsive, zero external requests. Skin shown: war-room dark. |

These are **style references for humans**, not runtime templates — the runtime catalog
stays in `Dashboards and Reports/Council/_templates/manifest.json` per Rule 6. The design
grammar both eras are measured against is `../design-system.md`.
