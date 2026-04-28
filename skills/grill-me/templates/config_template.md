# grill-me config

This file configures grill-me behavior for this project. All fields are optional. Read as natural language — not a strict schema.

## Default output target

What grill-me defaults to when asked "what is this feeding into?". One of: `prd`, `github_issue`, `spec`, `freeform`. If set, the skill skips the opening question.

Example: `default_output_target: github_issue`

## Output directory

Where grill-me writes session directories. Defaults to `docs/grill-me/`.

Example: `output_dir: docs/design/grills/`

## Standing dimensions

Things to always probe in this project. Each becomes an explicit branch the agent must open and resolve before the convergence pass can succeed. Free text, list as many as you want.

Example:
- Always ask about authentication and authorization implications
- Always ask about telemetry / observability surface
- Always ask about error budget impact
- Always ask about deployment surface (which services or regions ship this)

## Output template overrides

If you've placed custom templates in `.claude/grill-me/templates/`, list which ones here for reference. The skill auto-resolves them; this is just a note for humans reading the config.
