# Setting Up Playtest for Your Project

This guide walks you through customizing the `/mbriggs-skills:playtest` skill for your specific product.

## Quick Start (Minimal Setup)

If you just want to run a playtest with the generic personas:

1. Install the plugin: `/plugin install github.com/MarshallBriggs/mbriggs-skills`
2. Create a config file at `.claude/playtest/config.md` with your app's URL
3. Run `/mbriggs-skills:playtest`

That's it. The skill will use the built-in generic personas and evaluation framework.

## Full Customization

For the best results, create project-specific personas, competitive analysis, and evaluation criteria.

### Step 1: Create the override directory

```
.claude/playtest/
├── config.md                      # Required: app URLs and settings
├── evaluation_framework.md        # Optional: custom evaluation criteria
├── competitive_landscape.md       # Optional: competitor analysis
└── personas/                      # Optional: custom personas
    ├── your_persona_1.md
    ├── your_persona_2.md
    └── ...
```

### Step 2: Write config.md

This is the only required override file. Example:

```markdown
# Playtest Configuration

## App Name
My Awesome App

## Frontend URL
http://localhost:3000

## Backend URL
http://localhost:8080/api/health

## Reset Endpoint
POST http://localhost:8080/api/test/reset

## Output Directory
docs/playtests/runs/

## Real-World Scenario Test
A freelance designer has a client presentation in 2 hours. They need to create
a complete brand kit (logo variations, color palette, typography guide) using
our tool. Can they produce something they'd feel confident presenting?
```

### Step 3: Create custom personas (recommended)

Copy the persona template from `skills/playtest/templates/persona_template.md` and fill it in for your actual target users. If you create ANY persona files in `.claude/playtest/personas/`, they fully replace the plugin defaults — so include all the personas you want tested.

Tips:
- Start with 2-3 personas that represent your actual user segments
- Make task flows specific to YOUR product (not generic actions)
- Include real competitor names in frustrations
- The Voice section sets the tone — make it sound like a real person

### Step 4: Write competitive landscape (recommended)

Copy the template from `skills/playtest/templates/competitive_landscape_template.md` and populate it with real competitor data. This enables the skill to make meaningful competitive comparisons during playtests.

### Step 5: Customize evaluation framework (optional)

Only override the evaluation framework if you need domain-specific evaluation criteria beyond the generic 6-point framework. For most products, the default framework works well.

## How Overrides Work

The skill checks `.claude/playtest/` first, then falls back to plugin defaults:

| File | Override path | Behavior |
|---|---|---|
| Personas | `.claude/playtest/personas/*.md` | Full replacement (all or nothing) |
| Evaluation framework | `.claude/playtest/evaluation_framework.md` | Full replacement |
| Competitive landscape | `.claude/playtest/competitive_landscape.md` | Full replacement |
| Config | `.claude/playtest/config.md` | No default — always project-specific |

## Running a Playtest

```
/mbriggs-skills:playtest                           # asks all setup questions
/mbriggs-skills:playtest {persona_id}              # specific persona
/mbriggs-skills:playtest {persona_id} reactive     # reactive mode
```

Output goes to the configured output directory (default: `docs/playtests/runs/`).
