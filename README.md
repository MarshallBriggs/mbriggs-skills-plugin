# mbriggs-skills

A collection of Claude Code skills by Marshall Briggs.

## Installation

```
/plugin install github.com/MarshallBriggs/mbriggs-skills
```

## Skills

### playtest

Autonomous UX playtesting using Playwright MCP. Simulates user personas navigating your app in a real browser, captures screenshots, and produces structured feedback reports with bug documentation, UX evaluation, and competitive analysis.

**Requires:** [Playwright MCP server](https://github.com/anthropics/playwright-mcp) configured in your project.

**Quick start:**
```
/mbriggs-skills:playtest
```

**Features:**
- 4 built-in generic personas (new user, power user, impatient expert, complete beginner)
- Two test modes: scripted (fixed task list) or reactive (persona discovers tasks naturally)
- Project-local overrides for custom personas, evaluation criteria, and competitive analysis
- Structured output: per-task reports, UX debrief, bug rollup, SWOP analysis, priority action list
- Token-efficient: uses accessibility snapshots for navigation, screenshots only for reports

**Customization:**

The skill works out of the box with generic personas, but you'll get much better results by customizing it for your product. Templates are included to help you get started:

| What | Template | Override path |
|------|----------|---------------|
| App config (required) | — | `.claude/playtest/config.md` |
| Custom personas | [persona_template.md](skills/playtest/templates/persona_template.md) | `.claude/playtest/personas/*.md` |
| Competitive landscape | [competitive_landscape_template.md](skills/playtest/templates/competitive_landscape_template.md) | `.claude/playtest/competitive_landscape.md` |
| Evaluation framework | — | `.claude/playtest/evaluation_framework.md` |

See the full [project setup guide](skills/playtest/templates/project_setup_guide.md) for step-by-step instructions.

> **Tip:** If you run `/mbriggs-skills:playtest` without a config file, the skill will walk you through setup and offer to scaffold the override files for you.

## Adding Skills

Future skills will be added as new directories under `skills/`. Install the plugin once and new skills become available on update.

## License

MIT
