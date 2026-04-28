# mbriggs-skills

A Claude Code plugin providing reusable skills for UX testing and evaluation.

## Project Structure

```
.claude-plugin/
  plugin.json          # Plugin metadata (name, version, description, mcpServers)
skills/
  {skill-name}/
    SKILL.md           # Skill entry point (frontmatter + full orchestration)
    ...                # Supporting files (personas, frameworks, templates)
```

This repo holds the plugin itself. The marketplace listing lives separately at https://github.com/MarshallBriggs/mbriggs-skills.

Each skill is a self-contained directory under `skills/`. Everything a skill needs — personas, evaluation frameworks, templates, supporting docs — lives inside its own directory. No top-level shared directories.

## Adding a New Skill

1. Create `skills/{skill-name}/SKILL.md` with frontmatter (`name`, `description`) and full instructions.
2. Put all supporting files inside `skills/{skill-name}/`.
3. The skill is auto-discovered by Claude Code from the `SKILL.md` file.

## Conventions

- Skill directories use kebab-case.
- `SKILL.md` is always the entry point — never rename it.
- Skills should support project-local overrides via `.claude/{skill-name}/` in the consuming project when applicable.
- Templates and examples for end-user customization go in `skills/{skill-name}/templates/`.
- Zero runtime dependencies. Skills are pure markdown orchestration consumed by the Claude Code agent.

## Versioning

This plugin intentionally omits the `version` field in both `plugin.json` and the marketplace entry. Per the Claude Code marketplace docs, omitting `version` from a git-sourced plugin makes every commit on the default branch automatically register as a new version (using the commit SHA as the version identifier). Don't add `version` back — pushing is the release.

End users still need to manually click the refresh icon in the VSCode Marketplaces tab + the update button in the Plugins tab to pull the latest. The Claude Code VSCode extension has no auto-update toggle.

## Contributing

- Keep skills general-purpose. Domain-specific configuration belongs in project-local overrides, not in the plugin itself.
- Test skills against real applications before submitting.
- One skill per PR. Don't bundle unrelated changes.
- Fill in the PR description with what the skill does, how to invoke it, and what output it produces.
