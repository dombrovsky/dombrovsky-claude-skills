# Claude Code Skills

Personal collection of development workflow skills for Claude Code.

## Installation

```bash
claude skills install github:dombrovsky/dombrovsky-claude-skills
```

Or manually:
```bash
git clone https://github.com/dombrovsky/dombrovsky-claude-skills.git
cp -r dombrovsky-claude-skills/skills/git-atomic-workflow ~/.claude-code/skills/
```

## Skills

### git-atomic-workflow

Maintains clean git history using atomic commits, fixup workflow, and interactive rebase with autosquash.

**What it does:**
- Guides atomic commit creation (one logical change per commit)
- Creates fixup commits for small fixes to previous commits
- Uses `git rebase -i --autosquash` to clean up history
- Syncs feature branches with main using rebase (not merge)
- Detects unsafe scenarios (protected branches, multiple authors, existing merges)

**When to use:**
- Creating commits
- Fixing earlier commits
- Cleaning up history before PR
- Syncing branch with main

[Full Documentation](skills/git-atomic-workflow/README.md) • [Examples](examples/git-atomic-workflow-example.md)

## License

MIT
