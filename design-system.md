# Council Report Design System

This file is the visual contract every council report must satisfy, regardless of which
template the manifest selects. Templates are **skins**; this file is the **grammar**.
The manifest (`Dashboards and Reports/Council/_templates/manifest.json`) decides *which*
skin renders; this file decides *what every skin must contain and respect*.

Read this file together with the chosen template before rendering Step 5.

---

## Layer model

```
1. Tokens      — fixed names, template-defined values (the skin)
2. Components  — fixed structure, required in every report
3. Templates   — manifest-registered skins that restyle tokens, never restructure components
```

A template may change colors, typography flavor, density, and decoration. A template may
NOT remove a required component, rename an advisor identity color, or break a hard
requirement. If a template file conflicts with this spec, this spec wins.

---

## 1. Tokens

Every report defines these CSS custom properties on `:root`. The **identity and semantic
tokens are fixed values** — identical in every template so the reader builds recall across
reports. The **skin tokens are template-defined**.

### Advisor identity colors (fixed — never re-map)

| Token | Advisor | Value |
|---|---|---|
| `--adv-contrarian` | The Contrarian | `#e5484d` |
| `--adv-first-principles` | The First Principles Thinker | `#8e4ec6` |
| `--adv-expansionist` | The Expansionist | `#30a46c` |
| `--adv-outsider` | The Outsider | `#f5a623` |
| `--adv-executor` | The Executor | `#0091ff` |

If the roster changes, assign the new persona a new hue with AA contrast against both
light and dark surfaces, and record it here in the same commit that edits the roster.

### Verdict semantics (fixed)

| Token | Meaning | Value |
|---|---|---|
| `--sem-agree` | convergence / high-confidence | `#30a46c` |
| `--sem-clash` | genuine disagreement | `#f76808` |
| `--sem-blindspot` | caught only in peer review | `#8e4ec6` |
| `--sem-high` | HIGH magnitude/risk badge | `#e5484d` |
| `--sem-med` | MED badge | `#f5a623` |
| `--sem-low` | LOW badge | `#30a46c` |

### Skin tokens (template-defined)

`--bg`, `--surface`, `--surface-2`, `--border`, `--text`, `--text-muted`, `--accent`,
`--accent-contrast`. Dark or light is the template's choice; the contract is only that
every text/surface pairing passes WCAG AA.

### Type scale

- **Hero (the recommendation):** biggest type on the page, `clamp(26px, 4vw, 40px)`,
  weight 700+. Nothing else competes with it.
- Section headers: 18–22px. Body: 15–16px / 1.6 line height.
- Meta (timestamps, slugs, labels, badges): 11–13px, monospace, letter-spaced uppercase
  is encouraged.
- Spacing on a 4px grid.

### Polish floor (what keeps a report from looking flat)

Flat gray cards on a flat dark page fail the bar even if every component is present.
Every skin must bring:

- **Atmosphere** — the page background is layered (subtle radial glows, a faint grid or
  texture), never a single flat fill. Keep it quiet enough that text contrast never drops
  below AA, and strip it in `@media print`.
- **Depth** — cards get a soft drop shadow plus a 1px inner top highlight; key elements
  (hero rule, KPI top edge, advisor chips) carry a soft glow in their identity color.
- **Numerals that read like a scoreboard** — KPI values in mono, 22px+, weight 800, tinted
  to their semantic color. Stats are the second thing the eye lands on after the hero.
- **One signature moment** — each skin has a distinctive flourish (war-room: striped
  dissent segment + ghost section numerals; scoreboard: oversized tallies; terminal:
  prompt-style footer). One, not five.

---

## 2. Components (required in every report)

In reading order:

1. **Verdict hero** — the Chairman's recommendation, top of page, hero type, on `--accent`
   treatment. One sentence. A confidence chip next to it (derived from peer-review
   unanimity: UNANIMOUS / STRONG / SPLIT).
2. **First-step card** — the single concrete next action, visually paired with the hero,
   labeled `FIRST STEP`. One action, never a list.
3. **Vote split bar** — one horizontal bar, 5 equal segments, each filled with its
   advisor's identity color and grouped by stance (e.g., 4 phase / 1 cutover). Legend
   beneath with advisor name + stance keyword.
4. **Council positions grid** — one card per advisor: identity color chip, advisor name,
   lens (one line), stance (one line). Scannable in under ten seconds.
5. **Verdict sections** — Where the Council Agrees (`--sem-agree` edge), Where the Council
   Clashes (`--sem-clash` edge), Blind Spots the Council Caught (`--sem-blindspot` edge).
   Color is carried on a left border or header chip, not on body text.
6. **"What the council saw" panel** — the framed brief, clearly labeled, visually quieter
   than the verdict (the brief is context, not conclusion).
7. **Collapsible transcript** — full advisor responses and peer-review highlights inside
   native `<details>/<summary>` blocks, collapsed by default, summary row showing the
   advisor chip + name.
8. **Footer** — timestamp • slug • template id • "This is a recommendation, not a
   registered decision. To commit: fire `Decision: <X>` manually."

**Critical mode adds** (when `render_mode === "extended-analysis"`): a **KPI strip**
directly under the hero (confidence, vote split, regret asymmetry, reversibility cost —
4–5 single-stat cells, mono labels, big values) and the extended-analysis sections, each
using HIGH/MED/LOW badges from the semantic tokens.

---

## 3. Hard requirements

- **Single self-contained HTML file.** Inline CSS. Zero external requests — no CDN fonts,
  no JS libraries, no remote images. Reports must open offline, from any folder, years
  later. (System font stack or inline `@font-face` only.)
- **No required JavaScript.** Collapsing uses native `<details>`. Inline JS is allowed for
  garnish but the report must be fully readable with JS disabled.
- **WCAG AA contrast** for all text on its surface.
- **Responsive to 360px** — the grid stacks; nothing scrolls horizontally.
- **Print-safe** — `@media print`: expand all `<details>`, drop heavy backgrounds, keep
  identity colors as borders/chips so a grayscale print still reads.
- **No invented content.** Every component maps to a known pipeline source (brief, advisor
  responses, reviews, Chairman verdict, extended agents). Empty slot → render
  `*Not provided.*`, never fabricate.

---

## Changing this file

This spec lives in the skill repo and is versioned with it. Visual retunes of a single
template belong in the template file + manifest (no commit here). Changes to identity
colors, required components, or hard requirements belong here, with a commit message like
`design-system: add persona color for The Long-Termer`.
