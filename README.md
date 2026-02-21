# Claude Skills & Plugins

A personal collection of custom skills and plugins for Claude Code.

## Repository Structure

```
.
├── skills/          # Custom skills for Claude
├── docs/            # Documentation and guides
├── examples/        # Example implementations
└── shared/          # Shared utilities and helpers
```

## Getting Started

### Creating a New Skill

1. Create a new directory under `skills/` with your skill name
2. Add a `skill.json` manifest file
3. Implement your skill functionality
4. Document usage in the skill's README

### Installing Skills

Refer to the [Claude Code documentation](https://docs.claude.ai/claude-code) for instructions on installing custom skills.

## Available Skills

### git-atomic-workflow
Atomic commit workflow for feature branches with fixup commits, interactive rebase, and clean history maintenance.

**Features:**
- Atomic commit principles and guidelines
- Fixup commit workflow with autosquash
- Branch synchronization with rebase (not merge)
- Safety checks (protected branches, multiple authors, merge detection)
- PR review comment handling
- Conflict avoidance strategies
- Partial file staging for multiple fixups

[Documentation](skills/git-atomic-workflow/README.md) | [Examples](examples/git-atomic-workflow-example.md)

## Contributing

This is a personal repository, but feel free to fork and adapt for your own use.

## License

MIT