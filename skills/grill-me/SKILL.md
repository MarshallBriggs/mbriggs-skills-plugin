---
name: grill-me
description: Interview the user exhaustively about a feature, idea, or procedure before any design or implementation work begins. Walks the implicit design tree branch by branch, resolving dependencies between decisions. Convergence-driven — keeps grilling until a re-read of the conversation produces no new questions. Outputs a structured dossier with zero open questions, ready to convert into a PRD, GitHub issue, spec, or freeform context doc.
---

# Grill Me Skill

Interview the user relentlessly about a feature, idea, or procedure until every branch of the implicit design tree is resolved. Output a structured dossier with zero open questions that feeds directly into PRDs, GitHub issues, specs, or freeform context docs.

## When to use this skill

Use grill-me when:
- The user has an idea, feature, or procedure but the design has not been written down
- Before any meaningful design or implementation work begins
- When a brainstorm or plan keeps stalling on "wait, what about X?" gaps
- When a doc needs to be exhaustive enough to hand to a fresh engineer or paste directly into a GitHub issue

Do NOT use grill-me when:
- The user just wants to talk through an idea informally — use brainstorming
- The user already has a written spec and wants implementation steps — use writing-plans
- The user wants the agent to make the decisions for them — grill-me is interactive by design

## Content resolution order

This skill supports project-local overrides. For each customizable file, check these locations in order and use the FIRST one found:

1. **Project-local override**: `.claude/grill-me/{path}`
2. **Plugin default**: this skill's directory

### Templates
If `.claude/grill-me/templates/{name}.md` exists for a given output target, use it. Otherwise use the plugin default at `templates/{name}.md`.

### Config
Read `.claude/grill-me/config.md` if it exists. If absent, run the First-Time Setup flow before grilling anything.

## Project configuration

The optional `.claude/grill-me/config.md` file can contain any of the following. Read it as natural language — it is not a strict schema.

- **Default output target** — `prd` / `github_issue` / `spec` / `freeform`. If set, the skill skips the opening "what is this feeding into?" question.
- **Output directory** — defaults to `docs/grill-me/`.
- **Standing dimensions** — things to always probe in this project. Each becomes an explicit branch the agent must open and resolve before the convergence pass can succeed.
- **Output template overrides** — informational; templates auto-resolve from `.claude/grill-me/templates/`.

## First-time setup

This flow runs automatically when no `.claude/grill-me/config.md` is found in the project.

### Step A: Explain
Tell the user:
> This is your first grill in this project. I'll set up a config file so future grills are tuned to your project. After that, we can start grilling on your seed (or stop here so you can customize).

### Step B: Collect config
Ask in a single AskUserQuestion for:
1. **Default output target** — "What does most of your grilling feed into?" Options: PRD / GitHub issue / Spec / Freeform / Ask each time
2. **Output directory** _(optional)_ — defaults to `docs/grill-me/`
3. **Standing dimensions** _(optional)_ — free text. Anything always worth probing in this project (auth, telemetry, deployment surface, etc.)

Read `templates/config_template.md` from this skill's directory as a starting point. Write `.claude/grill-me/config.md` from the user's answers, using the template as the structure.

### Step C: Offer template scaffolding
Ask:
> I can also copy the four default output templates to `.claude/grill-me/templates/` if you want to customize them for this project. Want me to scaffold those?

If yes: copy `templates/prd.md`, `templates/github_issue.md`, `templates/spec.md`, `templates/freeform.md` from this skill's directory to `.claude/grill-me/templates/`.

### Step D: Proceed or stop
Ask whether to start a grill now (with the seed they passed in, if any) or stop and let them customize first.

## Usage

```
/mbriggs-skills:grill-me                                # asks for the seed
/mbriggs-skills:grill-me "<seed text>"                  # starts immediately with the given seed
/mbriggs-skills:grill-me <path-to-existing-doc>         # re-grill mode: fills gaps in the existing doc
```

---

## Orchestration flow

### Step 1: Setup

1. **Read config** — if `.claude/grill-me/config.md` exists, parse it. If not, run First-Time Setup.
2. **Read seed** — from the CLI argument if provided. If no argument, ask "what do you want to grill on?"
3. **Detect re-grill mode** — if the argument is a path to an existing file, branch to the Re-Grill flow at the bottom of this document.
4. **Lock the output target** — if config sets a default output target, use it. Otherwise ask in a single AskUserQuestion: "What is this feeding into?" with options PRD / GitHub issue / Spec / Freeform.
5. **Slugify the seed** — short kebab-case identifier (e.g., `sso-admin-panel`). Used in the session directory name.
6. **Create session directory** — `{output_dir}/YYYY-MM-DD-{slug}/`. Initialize `grill-log.md` with this header:

   ```markdown
   # Grill log: {slug}

   **Date:** YYYY-MM-DD
   **Seed:** {seed}
   **Output target:** {target}

   ## Decisions
   ```

### Step 2: Grill loop

Ask one question at a time. Pick style adaptively:

**Use AskUserQuestion when:**
- The decision has a small enumerable set (auth provider, DB engine, framework choice)
- The decision is binary or a small-N scope cut
- You can confidently predict 2-4 likely choices and "Other" covers the rest

**Use open prose when:**
- "Tell me more about X" / "walk me through Y"
- The decision space is genuinely open and pre-listing options would constrain the user
- The user's rationale matters as much as the answer
- Asking about edge cases, assumptions, or failure modes

When in doubt, use open prose. AskUserQuestion that misses the right option is worse than open prose that takes one extra round.

After every answer, append a bullet to `grill-log.md` under the `## Decisions` section:

```markdown
- **Q:** {question, paraphrased}
  - **A:** {answer, paraphrased}
  - **Spawned:** {any new follow-up questions this answer raised — leave blank if none}
```

Walk dependencies depth-first. If an answer says "we'll use SAML SSO," the next question is "via which IdPs?" not "what about logging?". Finish a branch before moving to siblings.

If standing dimensions are configured in `config.md`, treat each as a branch you must open and resolve. They do not all have to be the next question, but you cannot reach convergence without touching every one.

### Step 3: Convergence pass

When you believe no questions remain, do NOT silently start writing. Announce:

> "I think we're done. Here's what I considered closed:
> - [bulleted list of resolved branches, drawn from the grill log]
>
> I'm going to re-read our entire conversation and the grill log now and look for anything I missed. Push back on this list, name a branch I should still grill, or accept and I'll do the re-read."

Wait for the user's response. Three valid paths:
- **Accept** → proceed to silent re-read
- **Extend** → user names branches or specific questions to push on; back to Step 2 with those
- **Reject a "closed" branch** → user says branch X is not actually closed; reopen it in Step 2

Once accepted, do the silent re-read of:
- The full conversation (as much as is still in context)
- `grill-log.md` (covers anything compressed out of the conversation — this is the durable source of truth)
- The original doc, if in re-grill mode

Look specifically for:
- Contradictions between answers
- Assumptions made implicitly but never stated
- Dependencies named but not resolved
- Edges and failure modes that were skipped
- Scope statements without explicit out-of-scope counterparts
- Standing dimensions (if configured) that were never touched

Any new questions found → return to Step 2 with those questions. After grilling them, run convergence again.

A convergence pass that produces no new questions → proceed to Step 4. There is no fixed iteration cap.

### Step 4: Pre-write gap scan

Before writing the dossier, scan the in-memory draft for:
- **Literal strings**: `TBD`, `TODO`, `to be determined`, `we should figure out`, `still need to decide`, `not sure yet`, `unclear`
- **Structural**: any sentence in a decision section ending with `?` that is not quoting the user
- **Section**: an "Open Questions" or "Unknowns" section containing any items

Any hit → return to Step 2 with those gaps as the next questions. Re-run convergence after grilling them. The skill cannot exit with open questions in the dossier.

### Step 5: Write the dossier

1. Resolve the template path for the locked output target (project-local first, plugin default second).
2. Read the template.
3. Fill in every section using content from `grill-log.md` and the conversation.
4. Write to `{session_dir}/dossier.md`.
5. Re-run the gap scan on the written file. If any hits, return to Step 2.

### Step 6: After-doc message

Post a short message:

> "Dossier written to `{dossier_path}`. Grill log archived to `{grill_log_path}` in the same directory.
>
> Likely next steps:
> - Hand to writing-plans to turn this into an implementation plan
> - Open a GitHub issue from the dossier (if target was github_issue)
> - Send to brainstorming if more design exploration is needed
>
> Want me to do any of those, or are you good?"

Suggest only. Do NOT auto-invoke any next skill. The grill-me session ends after the user's response.

---

## Re-grill mode

Triggered when the CLI argument is a path to an existing file rather than a seed string.

### Step R1: Read the doc
Read the file in full. Treat it as the current state of resolved decisions.

### Step R2: Determine the output target
- If the doc has frontmatter with an `output_target` field, use that.
- Otherwise ask in a single AskUserQuestion: "What shape should the updated doc be?" with the four options.

### Step R3: Create re-grill session directory
`{output_dir}/YYYY-MM-DD-{slug}-regrill/`. Copy the original doc in as `original.md`. Initialize `grill-log.md` with a header noting this is a re-grill of `original.md` and the path to the original.

### Step R4: Initial gap scan
Apply the pre-write gap scan rules to the existing doc. Also flag:
- Assertions made without rationale ("we'll use X" with no "because Y")
- Scope statements with no explicit out-of-scope counterpart
- Named dependencies that were never resolved

For every gap found, write a bullet to `grill-log.md` under a `## Initial gaps` subsection.

### Step R5: Grill the gaps
Same loop as Step 2 above. Scoped to the gaps initially, but new questions can spawn new branches as usual. The convergence pass (Step 3) still applies, scoped to the union of: original doc + grill log + conversation.

### Step R6: Write the updated dossier
Same as Step 5 above. Writes to `{session_dir}/dossier.md`. Does NOT overwrite `original.md`. Tell the user where the new version is and that they can choose to replace the original if they want.

---

## Hard rules

- The skill cannot exit with open questions in the dossier. The gap scan is mechanical and non-negotiable.
- The skill cannot skip the convergence pass. Even with high confidence, the announced "I think we're done" + re-read is mandatory.
- The skill cannot auto-invoke writing-plans, brainstorming, or any other skill. After-doc behavior suggests only; the user decides.
- The skill must update `grill-log.md` after every answer. The log is the durable source of truth for the convergence re-read after context compression.
- One question per turn during the grill loop. Batches break the dependency-walking discipline.

## Token management

- The grill loop is conversational and can run long. Update `grill-log.md` after every answer so the convergence re-read works even after compression.
- The convergence re-read of `grill-log.md` is cheap (small file). The conversation re-read can be expensive — the log makes it less critical to fully re-read the conversation if context is short.
- Templates are small; reading them just before writing the dossier is fine.

## Skill package contents

**Plugin defaults:**

```
skills/grill-me/
├── SKILL.md                       ← orchestration (you are here)
├── templates/
│   ├── prd.md                     ← PRD output template
│   ├── github_issue.md            ← GitHub issue body template
│   ├── spec.md                    ← engineering spec template
│   ├── freeform.md                ← minimal context-doc template
│   └── config_template.md         ← scaffolded by first-time setup
```

**Project-local overrides** (take precedence over plugin defaults):

```
.claude/grill-me/
├── config.md                      ← output target, output dir, standing dimensions
└── templates/
    ├── prd.md                     ← optional, replaces plugin default
    ├── github_issue.md            ← optional
    ├── spec.md                    ← optional
    └── freeform.md                ← optional
```

**Run output** (default `docs/grill-me/`):

```
docs/grill-me/
└── YYYY-MM-DD-{slug}/
    ├── grill-log.md               ← running answers log, updated after every answer
    └── dossier.md                 ← final output, written after convergence + gap scan
```

Re-grill sessions live alongside as `YYYY-MM-DD-{slug}-regrill/` and additionally include `original.md`.
