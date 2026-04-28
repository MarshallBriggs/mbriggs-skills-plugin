# Playtest Evaluation Framework

This document directs the testing agent on *how* to evaluate the application during persona-based playtests. It contains the testing methodology, evaluation criteria, and cross-persona directives.

For competitor details and market positioning, see `competitive_landscape.md` if available.

---

## Critical Context for the Testing Agent

**You are evaluating a user-facing application.** The underlying architecture (backend framework, database, file system, APIs) should be invisible to the user. If at any point during testing the agent encounters raw technical details — implementation details, internal IDs, raw API errors, database terminology, file paths, or configuration syntax — as part of the *primary* user flow, that is a **critical UX failure** — not a rough edge to file for later, but a blocker that will lose target users immediately.

The primary interaction pattern the agent should be evaluating is: **the user describes what they need → the system produces relevant, context-aware output → the user reviews and acts on that output.** Everything else (data management, state, configuration) should emerge naturally from this core loop, not require separate management interfaces the user must maintain.

## The 6-Point Evaluation Framework

For each persona, the browser testing agent should:

1. **Adopt the persona fully.** Read the background, motivations, frustrations, and mental model. Every interaction with the app should be filtered through this persona's lens. Don't evaluate as an engineer — evaluate as this person.

2. **Attempt every task in the task flow list**, in order. These are ordered by the user's natural priority — what they'd try first, second, third when evaluating a new tool.

3. **Record friction at every step.** Friction includes: unclear labels, missing feedback, too many clicks, unexpected behavior, jargon the persona wouldn't know, features that seem missing, moments where the user would give up. **Pay special attention to any moment where the underlying implementation leaks through the UX** — implementation details, internal IDs, raw API errors, database terminology, file paths, or configuration syntax surfaced in the primary flow. These are the most damaging friction points for non-technical users.

4. **Evaluate quality of any generated or dynamic output.** For every piece of content the tool produces, ask: does this feel appropriate for the product context and the settings the user has established? Does it reference relevant context naturally? Does it feel like it was produced *for this user's specific situation*, not for a generic user? If output reads like filler content that could have come from any tool, that's a failure of the product's core differentiator.

5. **Note competitive comparisons.** For each task, consider: how does this compare to the same task in the competitor this persona is coming from? Faster? Slower? More confusing? More powerful? If `competitive_landscape.md` is available, use it for specifics. Otherwise note the general landscape as understood from the persona's background.

6. **Identify "aha moments."** Where does this tool do something the persona's previous tool couldn't? The most important aha moments are when the product demonstrates unique value — when output reflects context the user established earlier, when a workflow is dramatically shorter than expected, or when the tool anticipates a need the user hadn't explicitly stated.

## Bug Reporting Directive

**Bug documentation is persona-independent.** When anything breaks, errors, or behaves unexpectedly during a playtest, the testing agent must drop the persona lens and document the bug objectively, regardless of whether the current persona would notice or care about it.

Every bug report must include:
- **Exact steps to reproduce** — numbered, specific, starting from a known state
- **Expected behavior** — what should have happened
- **Actual behavior** — what actually happened
- **Screenshot** of the error state
- **Console errors or API failures** if visible in the browser dev tools or network tab

The persona's *reaction* to the bug (e.g., "This user would close the tab here") belongs in the task report's step-by-step narrative. But the bug report itself is clinical and complete — good enough for a developer to reproduce and fix without additional context.

Each task report has a dedicated `## Bugs Found` section. All bugs are also rolled up into SUMMARY.md for the full run.

## Holistic UX Debrief Criteria

After completing all tasks, the testing agent writes a holistic UX debrief (`ux_debrief.md`) covering the entire product experience — not just individual features. This debrief is written in the persona's voice with zero sugar-coating.

**Required sections:**

- **Overall impression** — "What is this tool? Could I explain it to a friend?" If the persona can't articulate what the product does after using it, that's a finding.
- **Emotional arc** — How did feelings change from first open to last task? Where excited, lost, annoyed, checked out?
- **Flow and coherence** — Does the app feel like one product or disconnected features?
- **Layout and visual hierarchy** — Does the screen make sense at a glance? Can the persona tell what's important vs. secondary? Is anything buried? Are related things grouped? Is the information density right or overwhelming?
- **Navigation and discoverability** — Could they find what they were looking for? Did they get lost? Are features hidden behind menus or icons they wouldn't click? Do buttons and labels make their purpose obvious?
- **Controls and affordances** — Do buttons look clickable? Do interactive elements give feedback? Are there elements that look clickable but aren't (or vice versa)? Are icons self-explanatory? Does the persona understand the difference between similar-looking options?
- **Accessibility** — Text readability, contrast, click target sizes, anything that makes the app harder to use. Not a formal audit — what the persona notices.
- **"Would I come back tomorrow?"** — Honest answer. Not "is the tech impressive" but "would I actually open this again." Specifically why or why not.
- **Unsolicited ideas** — Workflows the persona wishes existed, UX patterns from other tools that would help. Only high-impact ideas. When a suggestion would be clearer with a visual, include an ASCII mockup showing the proposed layout or flow.

**Critical:** Do not soften negative feedback. If the persona would hate something, say they hate it. Honest feedback is what improves the app. A glowing review that misses real problems is worse than useless.

---

## Persona Definitions

Full persona definitions (background, mental model, emotional state, frustrations, task flows with evaluation criteria) live in the resolved personas directory (see Content Resolution Order in SKILL.md).

---

## Cross-Persona Testing Directives

After testing with each individual persona, the agent should evaluate these cross-cutting concerns:

**Onboarding funnel**: Map the exact steps from "first open" to "first useful output" for each persona. Count clicks, count minutes, count moments of confusion. Compare onboarding speed across personas — power users should reach value faster than first-time users; adjust targets based on the persona profiles.

**Architecture leak audit**: Across all personas, catalog every instance where the underlying implementation leaks through the UX. This includes: raw file paths shown to the user, schema or configuration syntax visible in any non-optional view, API error messages surfaced without translation, references to internal system terminology in UI labels or tooltips, or any step that requires the user to understand the system's internal structure. For non-technical personas, each of these is a potential churn event. For power-user personas, some exposure may be acceptable — but it should be behind an "advanced" toggle, not in the primary flow.

**Feature discoverability**: For each persona, which features did they never find? Which features did they find but not understand? Which features did they understand but not find useful? A feature that exists but isn't discovered is the same as a feature that doesn't exist.

**Error states and dead ends**: For each task flow, deliberately make mistakes. Enter bad data. Leave required fields empty. Ask the system for something it can't handle. What happens? Does the tool recover gracefully or does the user hit a dead end? Dead ends are where users abandon tools permanently. Pay special attention to any generation or async failures — if the system returns poor output, does the user have a way to retry, refine, or manually edit? Or are they stuck?

**Real-world scenario test**: For every persona, simulate a realistic high-stakes usage scenario drawn from their background (see config.md for a product-specific scenario if provided; otherwise derive one from the persona's stated context and pressure). The question: using this tool, can the persona accomplish the thing they actually showed up to do — quickly enough, and well enough, that they'd feel confident in the result? This is the single most important test. Everything else is secondary to this.
