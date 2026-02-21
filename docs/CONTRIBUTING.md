# Contributing

This is a personal repository for custom Claude Code skills. While it's primarily for personal use, you're welcome to fork and adapt for your own needs.

## Using These Skills

Install all skills from this repository:
```bash
claude skills install github:dombrovsky/dombrovsky-claude-skills
```

Or fork and customize for your own workflow.

## Creating Your Own Skills

See [CREATING_SKILLS.md](CREATING_SKILLS.md) for a complete guide following the official [Agent Skills specification](https://agentskills.io/specification).

### Quick Start

1. Create a new directory under `skills/` with your skill name
2. Add `SKILL.md` with YAML frontmatter and instructions
3. Test the skill with Claude Code
4. Add to `.claude-plugin/marketplace.json` if publishing

### Skill Structure

```
my-skill/
├── SKILL.md          # Required: YAML frontmatter + instructions
├── README.md         # Optional: User documentation
├── scripts/          # Optional: Executable code
├── references/       # Optional: Detailed docs (loaded on demand)
└── assets/           # Optional: Templates, data, diagrams
```

## Pull Requests

This is a personal repo, but if you have improvements or bug fixes:

1. Fork the repository
2. Create a feature branch
3. Make your changes following the skill format
4. Test thoroughly
5. Submit a PR with clear description

## Questions or Issues

Open an issue for questions or to report problems with existing skills.

## License

MIT - See LICENSE file for details.
