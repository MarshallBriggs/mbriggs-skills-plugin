---
name: smoketest
description: Verify recent code changes work in a real browser. Analyzes the git diff, generates a tailored test checklist, and runs it via the bundled Playwright MCP server. Project-local config (frontend URL, optional backend health check, optional reset endpoint, output dir, base branch) lives at `.claude/smoketest/config.md` and is scaffolded on first run.
---

# Smoketest

Browser-based smoketest driven by what actually changed in the codebase. No hardcoded checklist — you generate the tests from the diff every time.

## Content resolution order

This skill supports project-local overrides. For each customizable file, check these locations in order and use the FIRST one found:

1. **Project-local override**: `.claude/smoketest/{path}`
2. **Plugin default**: this skill's directory

### Config
Read `.claude/smoketest/config.md` if it exists. If absent, run the First-Time Setup flow before running any smoketest.

## Project configuration

The required `.claude/smoketest/config.md` file can contain any of the following. Read it as natural language — it is not a strict schema.

- **Frontend URL** _(required)_ — where the app is served, e.g. `http://localhost:5173/`
- **Backend URL** _(optional)_ — health check endpoint, e.g. `http://localhost:8000/api/health`
- **Reset endpoint** _(optional)_ — POST endpoint or natural-language instructions to reset to a clean test state
- **Output directory** _(optional)_ — where to write smoketest run reports, defaults to `smoketest-results/`
- **Base branch** _(optional)_ — git branch to diff against for "what changed", defaults to `main`

## First-time setup

This flow runs automatically when no `.claude/smoketest/config.md` is found in the project.

### Step A: Explain
Tell the user:
> This is your first smoketest in this project. I'll set up a config file so future smoketests know which URLs to test against. After that we can run the smoketest.

### Step B: Collect config
Ask in a single AskUserQuestion for:
1. **Frontend URL** — required, e.g. `http://localhost:5173/`
2. **Backend URL** — optional, for health check
3. **Reset endpoint or instructions** — optional, for clean test state
4. **Output directory** — optional, defaults to `smoketest-results/`
5. **Base branch** — optional, defaults to `main`

Read `templates/config_template.md` from this skill's directory as a starting point. Write `.claude/smoketest/config.md` from the user's answers using the template as the structure.

### Step C: Proceed or stop
Ask whether to run the smoketest now or stop and let the user customize first.

## Prerequisites

- **Playwright MCP server** — bundled with this plugin and started automatically when `mbriggs-skills` is enabled. No manual `.mcp.json` setup required.

---

## Step 1: Analyze Changes

Read the base branch from config (defaults to `main`). Run these git commands to understand what changed:

```bash
# Uncommitted changes
git diff --stat
git diff --name-only

# Feature branch commits vs base
git log --oneline {base}..HEAD
git diff --stat {base}..HEAD
git diff {base}..HEAD -- '*.tsx' '*.ts' '*.jsx' '*.js' '*.vue' '*.svelte' '*.py'
```

From the diff output, identify:
- **New UI components** — need to verify they render and are interactive
- **Modified UI components** — need to verify changes work and nothing broke
- **New API endpoints** — need to verify the frontend successfully calls them
- **Changed data shapes** — need to verify the UI displays new/changed fields
- **New user interactions** — need to verify clicks, dropdowns, inputs work

Also discover available test selectors by grepping the codebase:

```bash
grep -rh 'data-testid="' --include='*.tsx' --include='*.ts' --include='*.jsx' --include='*.js' --include='*.vue' --include='*.svelte' | sed 's/.*data-testid="\([^"]*\)".*/\1/' | sort -u
```

These are the stable selectors you can use in Playwright snapshots and queries during Step 5.

## Step 2: Generate Test Checklist

Based on what you found in the diff, write a concrete test checklist. Each item should be:
- A specific user action ("click the Filter button on the items list")
- An expected result ("a dropdown opens showing field names with value counts")
- Testable in a browser with Playwright

Group tests by feature area. Prioritize happy paths — this is a smoketest, not exhaustive testing.

Present the checklist to the user before proceeding. Wait for confirmation.

## Step 3: Pre-Flight

Read URLs from config.

```bash
# Frontend (always check)
curl -s -o /dev/null -w "%{http_code}" {frontend_url}

# Backend (only if backend_url is configured)
curl -s -o /dev/null -w "%{http_code}" {backend_url}
```

Both must return `200`. If either fails, tell the user which service is not running (include the URL) and stop.

## Step 4: Set Up Test Environment

```bash
# Reset to clean state if a reset endpoint is configured
curl -s -X POST {reset_endpoint}

# Create output directory (gitignored)
mkdir -p {output_dir}/$(date +%Y-%m-%d-%H%M%S)
```

Save the output directory path — all screenshots and the report go there.

If a reset endpoint is configured but the call fails, warn the user and continue — the smoketest will still work, just not from a clean state.

If reset is configured as natural-language instructions instead of an endpoint, display them and wait for the user to confirm they have completed the reset before continuing.

If no reset config exists at all, skip this step entirely.

## Step 5: Run Tests

Navigate to the frontend URL using `mcp__playwright__browser_navigate`.

For each test in your checklist:
1. Perform the action using Playwright MCP tools
2. Take a screenshot (save to the output directory with a numbered descriptive filename)
3. Record PASS or FAIL with a short note

### Playwright Tips

- Use `browser_snapshot` to get the accessibility tree for element refs
- Use `browser_evaluate` with `document.querySelector` for elements without refs (find by `data-testid`, text content, or CSS selector)
- Use `browser_take_screenshot` after each test point
- Use `browser_resize` for responsive layout tests
- Use the `data-testid` list discovered in Step 1 as your stable selector pool
- If a test fails, note it and keep going. Don't stop on first failure.

## Step 6: Report

Write `report.md` to the output directory with:

```markdown
# Smoketest Report

**Date:** YYYY-MM-DD HH:MM
**Branch:** {branch name}
**Commit:** {short SHA + message}

## Results

| # | Test | Result | Screenshot | Notes |
|---|------|--------|------------|-------|
| 1 | ... | PASS/FAIL | 01-xxx.png | ... |

## Bugs Found
- (list any failures with details)

## Summary
X/Y tests passed.
```

Tell the user the results and where the report is saved.

---

## Skill package contents

**Plugin defaults:**

```
skills/smoketest/
├── SKILL.md                       ← orchestration (you are here)
└── templates/
    └── config_template.md         ← scaffolded by first-time setup
```

**Project-local config:**

```
.claude/smoketest/
└── config.md                      ← URLs, reset endpoint, output dir, base branch
```

**Run output** (default `smoketest-results/`):

```
smoketest-results/
└── YYYY-MM-DD-HHMMSS/
    ├── 01-{slug}.png
    ├── 02-{slug}.png
    ├── ...
    └── report.md
```
