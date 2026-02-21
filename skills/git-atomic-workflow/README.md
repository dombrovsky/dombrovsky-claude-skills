# Git Atomic Workflow

A Claude Code skill for an atomic commit workflow in feature branches using fixup commits, interactive rebase with autosquash, and clean history maintenance.

## Overview

This skill teaches Claude Code your git workflow principles:

- **Atomic commits**: Each commit represents one logical change
- **Clear messages**: Descriptive commit messages that explain what was done
- **Clean history**: Use fixup commits and autosquash to avoid noise
- **Preserved context**: Merge (not squash) to main to preserve development history
- **Safe rebasing**: Never rebase protected branches or multi-author branches

## Philosophy

Most git workflows fall into two camps:
1. Squash everything into one commit (loses valuable context)
2. Keep every tiny commit (creates noisy, hard-to-review history)

This workflow offers a middle ground: **preserve meaningful history while eliminating noise**.

By using `fixup!` commits and periodic interactive rebases, you maintain a natural development flow while ensuring the final history is clean and reviewable.

## Key Features

- **Autosquash workflow**: Small fixes get attached to their logical parent commits
- **Conflict-aware**: Guidelines for avoiding rebase conflicts with your own commits
- **Safety checks**: Never rebase protected branches or branches with multiple authors
- **PR-friendly**: Handles review comment fixes appropriately (fixup for minor, new commit for substantial)
- **Partial staging**: Support for creating multiple fixup commits from a single file's changes

## Installation

1. Copy this skill directory to your Claude Code skills location
2. Claude Code will automatically load it on next startup
3. The skill activates when working with git commits

## Usage

Once installed, Claude Code will automatically apply these principles when:

- Creating commits (ensures atomic, cohesive changes)
- Fixing previous work (uses fixup commits appropriately)
- Preparing branches for PR (runs interactive rebase with autosquash)
- Addressing PR comments (categorizes as fixup or substantial change)

### Manual Invocation

You can also explicitly reference the workflow:
- "Create a fixup commit for the authentication changes"
- "Clean up the branch history with rebase"
- "Commit this following clean history principles"

## Examples

### Scenario 1: Feature Development

```bash
# Initial feature work
git commit -m "Add user profile page layout"
git commit -m "Implement profile data fetching"

# Realize profile page needs CSS fix
git commit -m "fixup! Add user profile page layout"

# More feature work
git commit -m "Add profile edit functionality"

# Clean up before PR
git rebase -i --autosquash origin/main
# Results in 3 clean commits (fixup was squashed)
```

### Scenario 2: PR Review

```bash
# Original commits
git commit -m "Implement payment processing"
git commit -m "Add payment validation"

# PR reviewer: "Fix typo in error message"
git commit -m "fixup! Implement payment processing"

# PR reviewer: "Add support for refunds"
# This is substantial, deserves its own commit
git commit -m "Add refund processing to payment system"

# Clean up
git rebase -i --autosquash origin/main
```

## Command Examples

The skill includes examples of all relevant git commands:

- Branch safety checks
- Creating fixup commits
- Interactive staging with `git add -p`
- Running interactive rebase with autosquash
- Inspecting commit history
- Checking for multiple authors

See [skill.md](skill.md) for the complete command reference.

## Safety

The skill enforces important safety rules:

✅ **Use on feature branches** where you're the sole author
✅ **Rebase before pushing** or when certain no one else has the branch
✅ **Create meaningful commits** that tell a story

❌ **Never rebase** protected branches (main, master, develop)
❌ **Never rebase** branches with commits from multiple authors
❌ **Never rebase** after others have pulled your changes (without coordination)

## Contributing

This is a personal workflow skill. If you'd like to adapt it for your own workflow:

1. Copy the skill directory
2. Modify the principles in `skill.md`
3. Update triggers in `skill.json` if needed
4. Customize commit message guidelines

## License

MIT License - See repository root for details.
