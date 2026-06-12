# Extending the council

Read this file ONLY when editing the persona roster or judging whether a question merits a council. It is not needed during a normal run.

## When to run the council

The council is for questions where being wrong is expensive.

**Good council questions** (cross-domain — this fork is not marketing-specific):
- "Should I accept the EPMO secondment or stay on the Transformation track?"
- "Should I move my morning training to 5am or keep 6:30am and cut sleep elsewhere?"
- "Which of these three family-time redesigns keeps Fridays intact without hurting career velocity?"
- "Pitch the startup angle as Saudi-first or global-first?"
- "Replace the legacy auth middleware in one cutover or phase it over two quarters?"

**Bad council questions:**
- "What's the capital of France?" (one right answer)
- "Write me a tweet" (creation, not decision)
- "Summarize this article" (processing, not judgment)
- "Should I use markdown or rich text?" (trivial, no stakes)

The council tells you things you don't want to hear. That's the feature.

## Adding or modifying personas

The skill is Moe's fork — he owns it. To add, remove, or swap personas:

1. **Edit SKILL.md in three places:**
   - **"The five advisors" section** — add/remove/edit the persona description (1 short paragraph: what they optimize for, what tensions they create).
   - **Step 2 spawn count** — if the count changes (e.g., 5 → 7), update the prose to match. The prompt template is persona-agnostic and needs no change.
   - **Step 3 anonymization labels** — extend from A–E to A–<N-th letter>, and update the prose to "Spawn N reviewers, one per advisor."

2. **Keep the tension principle alive.** The 5-advisor roster works because it manufactures three natural tensions (Contrarian vs Expansionist, First Principles vs Executor, Outsider as wildcard). A new persona must create a NEW tension or sharpen an existing one — don't add a redundant angle.

3. **Commit to the fork** with a clear message like `add persona: The Long-Termer` or `replace Outsider with Domain Expert for Health decisions`.

4. **Future upgrade (not day-one):** a library of 15+ personas, with the skill dynamically picking 5–7 based on detected domain. Worth it once you've run ~20 councils and feel the fixed roster repeating itself. Seed it as a plant-seed when the pattern appears.

**Test your change** with a low-stakes question before running it on something real. If a persona produces generic or repetitive output in testing, tune the description or remove it.
