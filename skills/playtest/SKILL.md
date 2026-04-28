---
name: playtest
description: Run a UX playtest against any application frontend using a browser. Simulates a specific user persona navigating the app, captures screenshots, and produces structured feedback reports with bug documentation, holistic UX evaluation, and competitive analysis. Requires Playwright MCP server and a running frontend. Project-local personas, evaluation framework, and config override plugin defaults.
---

# UX Playtest Skill

Run a fully autonomous UX playtest of your application by adopting a specific user persona, navigating the real app in a browser, and producing structured markdown reports with screenshots, bug reports, and holistic UX evaluation.

## Content Resolution Order

This skill supports project-local overrides. For each content file (personas, evaluation framework, competitive landscape), check the following locations in order and use the FIRST one found:

1. **Project-local override**: `.claude/playtest/{file_path}`
2. **Plugin default**: this skill's directory (where this SKILL.md lives)

### Personas
If `.claude/playtest/personas/` exists and has `.md` files, use ONLY those personas (full replacement, not merge). Otherwise use personas from this skill's `personas/` directory.

### Evaluation Framework
If `.claude/playtest/evaluation_framework.md` exists, use it exclusively. Otherwise use this skill's `evaluation_framework.md`.

### Competitive Landscape
If `.claude/playtest/competitive_landscape.md` exists, use it exclusively. Otherwise use this skill's `competitive_landscape.md`.

### Project Config
Read `.claude/playtest/config.md` if it exists. This file provides project-specific settings (see Project Configuration below). If no config.md exists, run the First-Time Setup flow (see below) before proceeding.

## Project Configuration

The optional `.claude/playtest/config.md` file can contain any of the following. Read it as natural language — it's not a strict schema, just a markdown document with relevant info:

- **App name** — used in reports and persona descriptions
- **Backend URL** — health check endpoint (e.g., `http://localhost:8000/api/health`). Optional if no backend.
- **Frontend URL** — where the app is served (e.g., `http://localhost:5173/`). Required.
- **Reset endpoint** — optional API call to reset test state for a clean environment
- **Reset instructions** — optional manual steps to prepare a clean test environment
- **Output directory** — where to write playtest reports (default: `docs/playtests/runs/`)
- **Real-world scenario test** — a time-pressured realistic scenario that defines success for this product (e.g., "Can a user accomplish X within Y minutes under real conditions?"). Used in cross-persona evaluation.

## First-Time Setup

**This flow runs automatically when no `.claude/playtest/config.md` is found in the project.** It walks the user through creating their project-local override files so future playtests are properly configured.

### Step A: Explain what's needed

Tell the user:

> This is your first playtest in this project. I'll help you set up a config file and optionally create custom personas and a competitive landscape. This only needs to happen once — future runs will use the files we create now.
>
> At minimum, I need your app's frontend URL to run a playtest. But you'll get much better results if you also create personas that match your real target users and a competitive landscape document.

### Step B: Collect config information

Ask the user (in a single AskUserQuestion prompt) for:

1. **App name** — what's the product called?
2. **Frontend URL** — where is the app running? (e.g., `http://localhost:3000`)
3. **Backend URL** _(optional)_ — health check endpoint, if applicable
4. **Reset endpoint or instructions** _(optional)_ — how to reset to a clean state for testing
5. **Output directory** _(optional)_ — where to write playtest reports (default: `docs/playtests/runs/`)

Create `.claude/playtest/config.md` from their answers using the format described in Project Configuration above.

### Step C: Offer persona and competitive landscape scaffolding

After writing config.md, ask the user:

> I've created your config. Would you also like me to scaffold:
>
> 1. **Custom personas** — I'll copy the persona template to `.claude/playtest/personas/` so you can define your real target users. The built-in generic personas work for a first run, but custom personas produce much more useful feedback.
>
> 2. **Competitive landscape** — I'll copy the competitive analysis template to `.claude/playtest/competitive_landscape.md` so you can document your competitors. This enables competitive comparisons in playtest reports.
>
> You can do either, both, or neither. You can always add these later.

If they want personas: read `templates/persona_template.md` from this skill's directory and write it to `.claude/playtest/personas/persona_template.md`. Tell them to duplicate and fill in this file for each of their target user types.

If they want competitive landscape: read `templates/competitive_landscape_template.md` from this skill's directory and write it to `.claude/playtest/competitive_landscape.md`. Tell them to fill in the sections.

### Step D: Proceed or stop

Ask the user whether they want to run a playtest now (using the generic personas if they haven't filled in custom ones yet) or stop here and fill in their templates first. If they want to run, continue to Step 1 of the Orchestration Flow.

## Prerequisites

- **Playwright MCP server** — bundled with this plugin and started automatically when `mbriggs-skills` is enabled. No manual `.mcp.json` setup required.
- **Backend and frontend URLs** are read from project config (`.claude/playtest/config.md`) or collected during First-Time Setup

## Usage

```
/mbriggs-skills:playtest                                   # asks all setup questions at once
/mbriggs-skills:playtest {persona_id}                      # asks mode, environment, focus area
/mbriggs-skills:playtest {persona_id} reactive             # asks environment, focus area
/mbriggs-skills:playtest {persona_id} reactive "sessions"  # asks environment only
/mbriggs-skills:playtest {persona_id} scripted             # asks environment only
```

Available personas are discovered from the personas directory (project-local overrides first, then plugin defaults). Run `/mbriggs-skills:playtest` with no arguments to see the full list.

**Task Modes:**
- **Scripted** — follows the persona's hardcoded Task Flow from their persona file. Same tasks every run.
- **Reactive** — the persona generates tasks one at a time based on what they discover in the app. Each task emerges from the previous one. An optional focus area orients the exploration (e.g., "sessions", "onboarding"). No upfront task list — tasks are decided in the moment.

---

## Orchestration Flow

### Step 1: Parse Arguments & Setup (BLOCKING GATE)

Collect all required inputs. If ANY inputs are missing, ask for ALL missing inputs in a single AskUserQuestion prompt — never ask sequentially.

**If a persona ID is provided on the command line**, use it. Same for mode, focus area.

**Before presenting options**, read the resolved personas directory (project-local if it exists, otherwise plugin default) and build the persona list dynamically from the `.md` files found there. Present each persona with its name and a one-line summary drawn from the persona file.

**For any missing inputs**, present ONE AskUserQuestion prompt containing all of the following that are still needed:

1. **Persona** — numbered list with one-line descriptions, generated from the resolved personas directory

2. **Mode** — Scripted (fixed tasks from persona file) / Reactive (persona discovers tasks naturally as they go)

3. **Campaign environment** — Fresh start (reset to clean state per config) / Existing data (current state of the app). If no reset endpoint or reset instructions are configured, skip this option entirely and default to existing state.

4. **Focus area** _(optional, reactive mode only)_ — A theme or goal to orient the persona. Can be broad ("onboarding", "core workflow") or goal-oriented ("complete a specific end-to-end task"). Leave blank for fully open exploration. Ignored in scripted mode.

**Do NOT proceed to Step 2 until all inputs are collected.** No pre-flight checks, no browser actions, nothing until the user has answered.

### Step 2: Pre-Flight Checks

Before opening a browser:

1. **Check backend (if configured):** If a backend URL is specified in config.md, use Bash to `curl -s -o /dev/null -w "%{http_code}" {backend_url}`. Expect `200`. If it fails, tell the user the backend is not running (include the URL) and stop. If no backend URL is configured, skip this check.

2. **Check frontend:** Use Bash to `curl -s -o /dev/null -w "%{http_code}" {frontend_url}/`. Expect `200`. If it fails, tell the user the frontend is not running (include the URL) and stop.

3. **Apply environment choice** (already collected in Step 1):

   - If **Fresh start** and a **reset endpoint** is configured: Use Bash to POST to that endpoint. If the call fails, warn the user and continue — the playtest will still work, just not from a clean state.
   - If **Fresh start** and **reset instructions** are configured (but no endpoint): Display the reset instructions and wait for the user to confirm they've completed them before continuing.
   - If **Existing data** or no reset config: Skip this step entirely.

4. **Read evaluation framework:** Resolve the evaluation framework path per Content Resolution Order. Read the file. This contains the 6-point evaluation framework, cross-persona testing directives, and critical context about what constitutes a UX failure. Internalize this before starting the playtest. Every interaction you evaluate should be filtered through this lens.

   **Note:** The detailed competitor profiles and market gap analysis live in `competitive_landscape.md`. Do NOT read this during pre-flight — it may be large. Read it later when writing competitive comparisons in task reports and SUMMARY.md (Steps 5 and 8). If no competitive_landscape.md exists at either location, skip competitive comparisons throughout.

### Step 3: Load Persona

Resolve the personas directory per Content Resolution Order. Read the persona file for the selected persona.

Internalize:
- The persona's background, technical comfort, and mental model
- Their emotional state and what "success" means to them
- Their frustrations with existing tools (these inform what they'd compare the app to)
- The Voice section — think and react as this person throughout the entire playtest

**You are now this persona.** Every interaction with the app should be filtered through their perspective, not yours as an AI agent or engineer.

#### If Scripted Mode

Use the persona's **Task Flow** section as your ordered test plan. Proceed to Step 4.

#### If Reactive Mode

**Do NOT use the persona's Task Flow.** Do NOT generate an upfront task list.

The persona opens the app with no plan — just their background, their focus area (if provided), and what's on screen. They will decide what to do one task at a time based on what they discover. See Step 5 for the reactive execution loop.

If a focus area was provided, internalize it as a gravitational pull — the persona leans toward tasks related to this theme but isn't locked to it. If something more interesting or broken appears, they follow that instead.

### Step 4: Create Run Directory

Create the output directory using the output directory from config.md (default: `docs/playtests/runs/`):

```
{output_dir}/YYYY-MM-DD-{persona_id}-{NNN}/screenshots/
```

Where NNN is a zero-padded run number (001, 002, etc.). Check for existing directories with the same date and persona to determine the next number.

### Step 5: Execute Tasks

#### Scripted Mode

Work through every task in the persona's Task Flow, in order. For each task, follow the Navigation Strategy, 6-Point Evaluation Framework, Handling Dead Ends, Bug Reporting, and Task Report Writing sections below.

#### Reactive Mode

##### Working Scratch Pad

Create `notes.md` in the run directory at the start of the first task. Maintain it throughout the playtest with two sections:

```markdown
# Playtest Scratch Pad

## Noticed for Later
- (things spotted but not explored yet — each with a one-line note of why it's interesting)

## Open Bugs
- (any bug or broken behavior, documented regardless of whether the persona would care)
```

Update this file after every task. It survives context compression because it's on disk.

##### The Reactive Loop (repeats for each task)

1. **Observe** — Take an accessibility snapshot. What's on screen? What stands out? What's confusing?
2. **Decide** — As this persona, what do I want to try next? Informed by:
   - What just happened in the previous task (if any)
   - The scratch pad — is there something noticed earlier that's more interesting than what's in front of me right now?
   - The focus area (gravitational pull, not strict filter)
   - The persona's background, frustrations, and goals
   - What's visible on screen right now
3. **State the task** — Write a one-line goal in the persona's voice before starting (e.g., "I want to see if I can find that item I just created")
4. **Execute** — Navigate the app using the Navigation Strategy below. Apply the 6-Point Evaluation Framework. Follow Bug Reporting rules for anything that breaks.
5. **Write task report to disk** — Use the Task Report Template below. Write immediately after the task — don't accumulate.
6. **Update scratch pad** — Add anything new to "Noticed for Later," remove anything just explored, add any new bugs to "Open Bugs."

**Stopping condition:** 6-10 tasks (flexible). The persona stops when:
- They hit the upper bound (10), OR
- They've naturally run out of things they'd want to try within the focus area, OR
- They've hit enough dead ends that this persona would realistically give up and close the app

#### Navigation Strategy (Token Management)

- **Use accessibility snapshots** (`browser_snapshot`) as your primary way to understand what's on screen. These are cheap and give you the full DOM structure.
- **Fall back to visual screenshots** (`browser_screenshot`) only if the accessibility tree is too sparse to identify elements (e.g., missing ARIA labels, canvas-based UI).
- **Take real PNG screenshots for the report** at these moments:
  - Initial state before starting the task (what does the persona see?)
  - After key actions (what changed?)
  - Any confusion, error, or dead-end state
  - The final result
  - **Minimum 2 screenshots per task** (initial + final)

#### The 6-Point Evaluation Framework

Apply these to every task (from `evaluation_framework.md`):

1. **Adopt the persona fully.** Don't evaluate as an engineer. Evaluate as this person. If they wouldn't understand a label, that's friction. If they'd expect a button to be somewhere, note where they looked first.

2. **Record friction at every step.** Friction includes:
   - Unclear labels or icons
   - Missing feedback after actions
   - Too many clicks to accomplish something
   - Unexpected behavior
   - Jargon the persona wouldn't know
   - Features that seem missing
   - Moments where the user would give up and go back to their old tool
   - **CRITICAL: Any moment where the underlying architecture leaks through the UX** — implementation details, internal IDs, raw API errors, database schemas, file paths, or any technical terminology that wouldn't appear in a consumer-facing product. These are the most damaging friction points for non-technical personas.

3. **Evaluate generated output quality.** For every piece of content the tool generates, ask:
   - Does generated output match the product's established context and tone?
   - If the app generates content, is it contextually appropriate and consistent with what the user has set up?
   - Does it reference or connect to the user's existing data naturally?
   - If the output reads like generic filler content that ignores the user's context, that's a failure of the product's core value proposition.

4. **Note competitive comparisons.** For each task, consider: how does this compare to the same task in a competitor this persona might have used? Faster? Slower? More confusing? More powerful? Read `competitive_landscape.md` when you need specific competitor details for these comparisons. If no competitive_landscape.md exists, make general observations about comparable tools the persona would know.

5. **Identify "aha moments."** Where does the app do something that none of the competitors can? What makes this product uniquely valuable to this persona? Note any moment where the product demonstrates awareness of the user's context in a way that feels genuinely intelligent.

6. **Note feature discoverability.** Which features did the persona never find? Found but not understand? Understood but not find useful?

#### Bug Reporting

If anything breaks, errors, or behaves unexpectedly during execution: **drop the persona lens and document objectively.** Include:
- Exact steps to reproduce (numbered, starting from a known state)
- What happened vs. what should have happened
- Screenshot of the error state
- Console errors or API failures if visible

This goes in a dedicated `## Bugs Found` section in the task report. The persona's *reaction* to the bug still goes in the step-by-step section (e.g., "The persona would close the tab here"), but the bug report itself is clinical and complete — good enough for a developer to reproduce and fix.

#### Handling Dead Ends and Missing Features

Many tasks will target features that don't exist yet. **This is expected and valuable — gap analysis is the point.**

When a feature doesn't exist, a button errors, or the app crashes:
- **Do NOT retry or get stuck.** Document it as a finding (and as a bug if applicable).
- Take a screenshot of the current state.
- Record what the persona *expected* to find vs what was actually there.
- Note the emotional impact: "This is where this persona gives up and goes back to their old tool."
- Consider: what would a relevant competitor offer for this task? (Refer to competitive_landscape.md if available.)
- Move to the next task.

#### Writing the Task Report

After completing (or hitting a dead end on) each task, immediately write the report to disk. Don't accumulate multiple tasks in memory.

Save to: `{output_dir}/{run_dir}/task_{N}_{brief_slug}.md`

The slug is derived from the first few words of the task heading (lowercased, underscored). Example: `task_1_first_launch_onboarding.md`

### Step 6: Cross-Persona Evaluation

After completing all tasks, write a cross-cutting evaluation section (this goes in SUMMARY.md):

- **Onboarding funnel:** Map the exact steps from "first open" to "first useful output." Count clicks, count minutes, count moments of confusion. Evaluate against targets in config.md if specified; otherwise assess whether time-to-value is reasonable for each persona type based on their technical comfort level and expectations.

- **Architecture leak audit:** Catalog every instance where the underlying implementation leaked through the UX — internal architecture, database schemas, file paths, API errors, internal IDs, configuration formats, or any implementation detail that end users shouldn't see.

- **Feature discoverability:** Which features were never found? Found but not understood? Understood but not useful?

- **Error states and dead ends:** List all dead ends encountered. For each, note: was there a graceful recovery path, or was the user stuck?

- **Real-world scenario test:** Use the scenario from config.md if provided. Otherwise, simulate a realistic time-pressured use case: the persona has a real deadline and limited time. Can they accomplish their core goal using this tool? This is the single most important test — not "is the tech impressive" but "does this actually work for a real person under real pressure."

### Step 7: Write UX Debrief

Save to: `{output_dir}/{run_dir}/ux_debrief.md`

After completing all tasks, the persona sits back and reacts to the product as a whole. This is NOT a summary of individual task findings — it's a holistic assessment of the entire experience. Written in the persona's voice with zero sugar-coating.

**Sections:**

1. **Overall impression** — "What is this tool? Could I explain it to a friend?" If the persona can't articulate what the product does after using it, that's a finding.
2. **Emotional arc** — How did their feelings change from first open to last task? Where excited, where lost, where annoyed, where checked out?
3. **Flow and coherence** — Does the app feel like one product or disconnected features? Do the pieces connect in a way that makes sense to this person?
4. **Layout and visual hierarchy** — Does the screen make sense at a glance? Can the persona tell what's important vs. secondary? Is anything buried? Are related things grouped together? Is the information density right or overwhelming?
5. **Navigation and discoverability** — Could they find what they were looking for? Did they get lost? Are features hidden behind menus or icons they wouldn't think to click? Do buttons and labels make their purpose obvious without hovering or guessing?
6. **Controls and affordances** — Do buttons look clickable? Do interactive elements give feedback when used? Are there elements that look clickable but aren't (or vice versa)? Are icons self-explanatory or do they need labels? Does the persona understand the difference between similar-looking options?
7. **Accessibility** — Text readability, contrast, click target sizes, anything that would make the app harder to use for someone with visual or motor difficulties. Not a formal audit — just what the persona notices.
8. **"Would I come back tomorrow?"** — Honest answer. Not "is the tech impressive" but "would I actually open this again." Specifically why or why not.
9. **Unsolicited ideas** — Things the persona wishes existed, workflows they expected, UX patterns from other tools that would help here. Only high-impact ideas that would improve the app by a strong margin — user flow, experience, or accessibility. When a suggestion would be clearer with a visual, include an ASCII mockup showing the proposed layout or flow.

**Critical:** Do not soften negative feedback. If the persona would hate something, say they hate it. If they'd be confused, say confused. Honest feedback is what improves the app. A glowing review that misses real problems is worse than useless.

### Step 8: Write SUMMARY.md

Save to: `{output_dir}/{run_dir}/SUMMARY.md`

SUMMARY.md is the one file to read for the full picture of the playtest. Every section is a summary with pointers to the detailed files (task reports, `ux_debrief.md`, `notes.md`) for deeper reading.

Include these sections:

1. **Run metadata** — persona name, date, mode (scripted/reactive), focus area (if any), number of tasks completed, total time
2. **UX Debrief Summary** — condensed key findings from `ux_debrief.md`: overall impression in 2-3 sentences, emotional arc highlights, layout/navigation/controls callouts, accessibility notes, and the "would I come back tomorrow" verdict
3. **SWOP Analysis** (Strengths / Weaknesses / Opportunities / Problems):
   - **Strengths:** What works exceptionally well? What features define the app's value?
   - **Weaknesses:** Where does the UI fail to communicate its intent? What is clunky or unintuitive?
   - **Opportunities:** What missing features or small tweaks could exponentially improve the experience?
   - **Problems:** What critical bugs or severe UX anti-patterns actively block the user from succeeding?
4. **Competitive positioning** — How do our findings map against the market? Where do we beat competitors? Where do they beat us? What's our clearest differentiator from this persona's perspective? Only include this section if `competitive_landscape.md` is available at either resolution path.
5. **Cross-persona evaluation results** (from Step 6)
6. **Bugs Found** — rollup of all bugs from every task report, deduplicated, each with repro steps. Every bug entry should be self-contained — a developer should be able to reproduce and fix it from this section alone.
7. **Unsolicited Ideas** — high-impact suggestions from the persona's UX debrief. When a suggestion involves layout or visual design, include an ASCII mockup showing the proposed change.
8. **Priority action list:**
   - **Top NEEDS:** Mission-critical updates required for basic feature completion (broken endpoints, crashes, invisible UI states, architecture leaks, critical bugs)
   - **Top WANTS:** Quality-of-life improvements that elevate from "functional" to "polished" (shortcuts, animations, extra generation buttons, flow improvements)

---

## Task Report Template

Use this template for every task report:

```markdown
# Task {N}: {Task Name}

## Role & Goal

**Persona:** {persona name}
**Objective:** {what the persona is trying to accomplish, in their words}
**Competitor baseline:** {what a relevant competitor offers for this task — omit this field if no competitive_landscape.md is available}

## Step-by-Step Experience

### Step {N}: {action description}

- **Expectation:** What did the user think was going to happen?
- **Reality:** What actually happened?
- **Thought / Feeling:** Was it intuitive, confusing, or delightful? Use the persona's voice.
- **Screenshot:** ![description](screenshots/{filename}.png)

[Repeat for each significant interaction]

## Competitive Comparison

How does this experience compare to a relevant competitor for the same task?
Is our output meaningfully better? If not, why would this user switch?
(Omit this section if no competitive_landscape.md is available and no meaningful comparison can be made.)

## Bugs Found

[If no bugs were found during this task, write "No bugs found during this task."]

### Bug {N}: {short description}

- **Steps to reproduce:**
  1. {exact step from a known state}
  2. {next step}
  3. ...
- **Expected:** {what should have happened}
- **Actual:** {what actually happened}
- **Screenshot:** ![bug description](screenshots/{filename}.png)
- **Console errors:** {any errors visible in dev tools, or "None observed"}

[Repeat for each bug]

## Summary Verdict

- **Likes:** What worked well from this persona's perspective
- **Dislikes:** What frustrated them or felt broken
- **Architecture leaks:** Any moments where implementation details surfaced in the UI
- **Aha moments:** Any moments where the app did something competitors can't
- **Would this persona continue?** Yes/No — and specifically why
```

---

## Token Management

This skill is token-intensive due to screenshots. Follow these rules:

1. **Accessibility snapshots for navigation** — use `browser_snapshot` to understand what's on screen before deciding what to click. This is cheap (~200-500 tokens).

2. **Real screenshots only for the report** — use `browser_screenshot` at the moments listed above (initial state, after action, confusion/error, final result). Each screenshot costs ~1500-3000 tokens.

3. **Write to disk immediately** — after each task, write the report file. Don't keep multiple task transcripts in memory.

4. **Summarize completed tasks** — if context is getting long, summarize what you've already written to disk and release the detailed memory. You can always re-read the files.

5. **Estimated budget:** A full persona run (7-8 tasks) will use approximately 50-100k tokens and take 15-30 minutes.

---

## Skill Package Contents

This skill supports both plugin-level defaults and project-local overrides.

**Plugin defaults** (used when no project-local overrides exist):

```
skills/playtest/
├── SKILL.md                    ← You are here (orchestration + report template)
├── evaluation_framework.md     ← 6-point framework + cross-persona directives (loaded at pre-flight)
├── competitive_landscape.md    ← Competitor profiles + market gap analysis (optional, loaded when writing comparisons)
├── personas/
│   └── {persona_id}.md         ← One file per persona
├── templates/
│   ├── competitive_landscape_template.md  ← Template for project-local competitive analysis
│   ├── persona_template.md                ← Template for creating custom personas
│   └── project_setup_guide.md             ← Guide for setting up playtests in a project
```

**Project-local overrides** (take precedence over plugin defaults, project-specific):

```
.claude/playtest/
├── config.md                   ← App name, URLs, reset config, output dir, real-world scenario test
├── evaluation_framework.md     ← Replaces plugin evaluation_framework.md if present
├── competitive_landscape.md    ← Replaces plugin competitive_landscape.md if present
├── personas/
│   └── {persona_id}.md         ← Replaces entire plugin personas/ directory if any files exist here
```

Run output goes to the configured output directory (default: `docs/playtests/runs/`) — each run produces task reports, `notes.md` (scratch pad), `ux_debrief.md`, and `SUMMARY.md`.
