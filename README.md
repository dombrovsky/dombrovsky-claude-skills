# Claude Code Skills

Personal collection of development workflow skills for Claude Code.

## Installation

```bash
claude skills install github:dombrovsky/dombrovsky-claude-skills
```

Or manually:
```bash
git clone https://github.com/dombrovsky/dombrovsky-claude-skills.git
cp -r dombrovsky-claude-skills/skills/* ~/.claude-code/skills/
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

### csharp-code-cleanup

Automated C# code cleanup using ReSharper CLI to fix StyleCop violations and maintain consistent code style. **C# ONLY**.

**What it does:**
- Fixes StyleCop violations (SA#### errors) automatically
- Formats C# code using ReSharper cleanup profiles
- Uses project's `.DotSettings`, `.editorconfig`, and `stylecop.json` configuration
- Detects violations from build output and offers to fix them
- Supports modified files, specific files, or entire solution cleanup

**When to use:**
- Build fails with StyleCop violations
- Before committing C# code changes
- After modifying C# files to ensure consistent style
- When you see SA#### errors in build output

**Commands:**
- `/csharp-cleanup` - Clean up modified C# files
- `/csharp-cleanup --all` - Clean up entire solution
- `/csharp-cleanup --verify` - Preview changes without applying
- `/resharper-cleanup` - Alias for csharp-cleanup
- `/stylecop-fix` - Alias for csharp-cleanup

[Full Documentation](skills/csharp-code-cleanup/README.md)

## License

MIT
