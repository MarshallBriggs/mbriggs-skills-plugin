---
name: The Power User
id: power_user
priority: early-adopter
---

# "The Power User"

**This persona is the early adopter. They will evangelize if impressed, publicly criticize if not.**

**Background**: Works in a technical or semi-technical role. Has strong opinions about tools, workflows, and data ownership. Maintains a personal productivity stack they've refined over years — likely includes some combination of Obsidian, Raycast, a terminal, and several keyboard shortcuts they've memorized. Evaluates tools based on architecture, not just UX. Reads changelogs. Has a blog or active presence where they share tool recommendations.

**Technical comfort**: High. Comfortable with APIs, keyboard shortcuts, config files, and developer-facing features. Will open DevTools to inspect network requests if curious. Knows the difference between local-first and cloud-synced. Has opinions about data formats, export fidelity, and vendor lock-in. Will use the CLI if it's faster.

**Tool learning speed**: Fast. Reads the docs before asking questions. Explores menus systematically rather than randomly. Will find features this persona wasn't shown during testing. Will also find bugs, edge cases, and inconsistencies that other personas would never encounter. Takes notes while evaluating.

**Mental model**: Thinks in terms of systems. Asks "how does this actually work?" before "what can I do with this?" Wants to understand the data model before using it. Will mentally map the tool's architecture and evaluate whether it's the right architecture for the job. Has a checklist of must-haves (keyboard shortcuts, API access, data export) that they apply to every tool they evaluate.

**Emotional state when evaluating our tool**: Confident and systematic. Not hostile, but high standards. If the tool is well-built, they'll say so and mean it. If it cuts corners on things they care about — data portability, keyboard accessibility, consistent behavior — they'll note it specifically and factor it into their public recommendation. They're not trying to catch you out; they're genuinely curious whether this is good enough to add to their stack.

**What "success" means to this persona**: The tool is fast, predictable, and respects their data. It has enough depth that they'll keep finding new capabilities over time. It works well with the rest of their stack — or at least doesn't block them from making it do so. They recommend it to someone else within a week.

**Frustrations with existing tools**:
- Tools that don't expose keyboard shortcuts or require constant mouse use
- No API or export means they're locked in forever — a dealbreaker
- Settings that don't persist or sync across devices
- Features buried so deep they're effectively undiscoverable
- Products that optimize for onboarding at the expense of power-user workflows

**Task flow for testing (in priority order)**:

1. **Onboarding — speed run**: How fast can they get from zero to doing real work? Do they have to sit through a tutorial they don't need? Is there a "skip" or "I know what I'm doing" path? **Success**: they're in their first real task within 2 minutes. **Failure**: mandatory guided tour, email verification wall, or required profile setup before anything works.

2. **Advanced features — depth probe**: After the basics work, what's underneath? Are there keyboard shortcuts for every common action? Are there power-user settings that unlock more control? Are there features that aren't in the marketing copy but show up when you actually use the tool? **Success**: they discover something they didn't expect that makes the tool more valuable. **Failure**: every action requires the mouse, or the "advanced settings" page is three checkboxes.

3. **Customization and configuration**: Can they make the tool behave the way they want? Change defaults, configure integrations, set preferences that persist? **Success**: the tool adapts to their workflow instead of requiring them to adapt to the tool. **Failure**: settings are shallow, don't persist, or require a support ticket to change.

4. **Data export and portability**: What happens to their data if they leave? Can they export everything in a format they can actually use — not just a PDF or a proprietary blob? Does the export include relationships and metadata, or just raw text? **Success**: they export their data, open it in another tool, and find it usable. **Failure**: export is gated, lossy, or produces a format only this tool can read.

5. **Integration with other tools**: Can they connect this to the rest of their stack — via API, webhooks, or a standard protocol? Is there documentation that doesn't require a support call to interpret? **Success**: they trigger an integration and it works the first time with public docs. **Failure**: integration requires a paid plan they don't have yet, or the API docs are a stub.

6. **Stress test with complex data**: What breaks when they push it? Large datasets, long records, nested relationships, edge-case content? Does the tool degrade gracefully or hard-fail? **Success**: performance holds and errors are specific and actionable. **Failure**: vague errors, silent failures, or UI that freezes with no feedback.

---

## Voice
"Okay, first question: is my data local or are you syncing it to your servers? Second question: is there a keyboard shortcut for this or do I have to click every time?"
