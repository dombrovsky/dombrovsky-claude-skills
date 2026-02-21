---
name: git-atomic-workflow
description: Use this skill when committing code changes, creating commits, using git, rebasing branches, or syncing with main/master. Provides atomic commit workflow for feature branches using fixup commits and interactive rebase with autosquash. Maintains clean git history while preserving meaningful development context. Use when the user mentions commits, git, rebasing, fixing history, or syncing branches.
license: MIT
metadata:
  author: Volodymyr Dombrovskyi
  version: "1.0.0"
---

# Git Atomic Workflow

Atomic commit workflow for feature branches using fixup commits, interactive rebase with autosquash, and clean history maintenance.

## Core Principles

### 1. Atomic, Cohesive Commits
- Each commit should represent a single logical change
- Commit messages should clearly describe what was done
- Avoid commits that bundle multiple unrelated changes together
- Commits should be atomic: reverting one commit should only remove that specific aspect
- Ideally, checking out any commit in the middle of a feature branch should result in compilable, working code

### 2. Preserve History, Not Noise
- Feature branches are merged to main using **merge commits** (not squash-merge)
- This preserves the full feature development history for future reference
- However, the preserved history must be clean and meaningful
- Low-value commits like "fix: typo", "fix PR comments" should not appear in final history

### 3. Autosquash Workflow
- Use `fixup!` commits to attach fixes to previous commits
- Periodically run interactive rebase with autosquash to clean up history
- This keeps history clean while maintaining a natural development flow

### 4. Branch Synchronization with Base
- Keep feature branch synchronized with base branch (main/master/develop) using **rebase, NOT merge**
- Exception: If the branch already contains merge commits, do NOT rebase it
- Exception to exception: If rebasing from a point at or after the last merge commit, rebase is safe
- Rebase keeps linear history which makes it easier to review and understand the feature development

**Why avoid merge commits in feature branches:**
- Merge commits from base branch create noise in feature history
- They make it harder to see the actual feature development
- Interactive rebase becomes complicated when merge commits are involved
- Linear history is cleaner and easier to review

**When merge commits exist:**
- Check if branch contains any merge commits: `git log --oneline --merges HEAD ^origin/main`
- If it does, either continue with merges OR rebase from after the last merge commit
- Don't try to rebase over merge commits unless absolutely necessary

## Quick Start

### Basic Workflow

```bash
# Create feature branch
git checkout -b feature/new-feature

# Make commits
git commit -m "Add feature component"
git commit -m "Add feature tests"

# Need to fix first commit
git commit -m "fixup! Add feature component"

# Clean up before PR
git rebase -i --autosquash $(git merge-base origin/main HEAD)
```

### Common Commands

```bash
# Create fixup commit for previous work
git commit -m "fixup! Original commit message"

# Check for merge commits before syncing
git log --oneline --merges HEAD ^origin/main

# Sync with main (no merges)
git rebase origin/main

# Interactive rebase with autosquash
git rebase -i --autosquash <base-commit>
```

## Detailed Workflows

### Creating Commits

**For new logical changes:**
```bash
git add <files>
git commit -m "Clear, concise description of what was done"
```

**For fixes to the last commit:**
```bash
git add <files>
git commit --amend --no-edit
# Or with message update:
git commit --amend -m "Updated message"
```

**For fixes to earlier commits:**
```bash
# Find the commit message you want to fix
git log --oneline

# Commit with fixup prefix
git add <files>
git commit -m "fixup! Original commit message"
```

### Partial File Staging (Multiple Fixups from One File)

When working directory contains changes belonging to multiple previous commits, use interactive staging:

**Example scenario:**
You have file `src/auth.js` with unstaged changes:
- Lines 10-15: Bug fix for "Add login functionality" commit
- Lines 45-50: Enhancement for "Add logout functionality" commit

**Actual file diff:**
```diff
diff --git a/src/auth.js b/src/auth.js
--- a/src/auth.js
+++ b/src/auth.js
@@ -10,7 +10,10 @@ function login(username, password) {
   if (!username || !password) {
-    return null;
+    throw new Error('Username and password required');
   }
+  if (username.length < 3) {
+    throw new Error('Username too short');
+  }
   return authenticate(username, password);
 }
 
@@ -45,6 +48,9 @@ function logout(sessionId) {
   if (!sessionId) {
     return false;
   }
+  // Clear user session data
+  clearSessionData(sessionId);
+  
   return destroySession(sessionId);
 }
```

**Workflow:**
```bash
# View the diff first
git diff src/auth.js

# Start interactive staging
git add -p src/auth.js

# Git shows first hunk (lines 10-15, login validation):
# @@ -10,7 +10,10 @@ function login(username, password) {
# Stage this hunk [y,n,q,a,d,j,J,g,/,s,e,?]? y

# Git shows second hunk (lines 45-50, logout cleanup):
# @@ -45,6 +48,9 @@ function logout(sessionId) {
# Stage this hunk [y,n,q,a,d,j,J,g,/,s,e,?]? n

# Commit first fixup (only login changes staged)
git commit -m "fixup! Add login functionality"

# Now stage the remaining logout changes
git add -p src/auth.js
# Stage this hunk [y,n,q,a,d,j,J,g,/,s,e,?]? y

# Commit second fixup
git commit -m "fixup! Add logout functionality"
```

**Interactive staging options:**
- `y` - stage this hunk
- `n` - do not stage this hunk
- `s` - split the hunk into smaller hunks (if possible)
- `e` - manually edit the hunk
- `?` - show help

**This is critical**: Don't bundle unrelated fixes into one commit just because they're in the same file. Proper fixup targeting keeps commits atomic.

### Running Interactive Rebase

Periodically clean up fixup commits. **IMPORTANT**: Only run rebase when user explicitly asks for it.

```bash
# Find the base commit (one before your first commit that needs squashing)
git log --oneline

# Rebase from the commit BEFORE the first one you want to squash
git rebase -i --autosquash <base-commit-hash>^

# Or rebase from branch point (common base with main)
git rebase -i --autosquash $(git merge-base origin/main HEAD)

# Or specify number of commits if you know the count
git rebase -i --autosquash HEAD~10
```

**IMPORTANT: Choosing the correct rebase base:**
- Use the commit **before** the first one that has fixup commits
- OR use the branch base (where you branched from main)
- **DO NOT** blindly rebase to latest origin/main unless user asks for it and you have verified no merge commits exist
- **DO NOT** run interactive rebase automatically - only when user requests it

The autosquash option automatically:
- Reorders fixup! commits to appear after their target commits
- Marks them for squashing during the interactive rebase

### Branch Synchronization

When you need to update your feature branch with latest changes from base branch:

**Step 1: Check for merge commits**
```bash
git fetch origin
git log --oneline --merges HEAD ^origin/main
```

**Step 2: Choose strategy based on result**

If **NO merge commits** (empty output):
```bash
# Safe to rebase - keeps linear history
git rebase origin/main
```

If **HAS merge commits**:
```bash
# Option 1: Continue using merge strategy
git merge origin/main

# Option 2: Rebase from after the last merge commit
LAST_MERGE=$(git log --oneline --merges HEAD ^origin/main | head -1 | cut -d' ' -f1)
git rebase --onto origin/main $LAST_MERGE HEAD
```

**Why this matters:**
- Rebasing over merge commits can cause complex conflicts and duplicate commits
- Once you introduce a merge commit, the branch history changes structure
- Consistent strategy prevents confusion and conflicts

## Safety Checks

### NEVER use this workflow when:

**1. On protected branches** (main, master, develop, etc.)
**2. When multiple authors have commits on the branch**
**3. After the branch has been pushed and others might have pulled it**
**4. DO NOT run interactive rebase automatically** - Only suggest running rebase when appropriate

## Commit Message Guidelines

### Good commit messages:
- "Add user authentication with JWT tokens"
- "Refactor database connection pooling logic"
- "Fix race condition in payment processing"
- "Update API endpoint to support pagination"

### Poor commit messages (should be fixups):
- "fix typo"  
- "PR comments"
- "oops"
- "fix tests"
- "minor changes"

## PR Comment Fixes

When addressing PR review comments:

- **Substantial changes** that deserve their own place in history to Regular commit with good message
- **Small fixes, typos, style changes** to fixup! commit referencing the original commit being fixed

Example:
```bash
# PR reviewer says: "Add input validation to the login function"
# This was part of commit "Add user authentication with JWT tokens"
git add src/auth/login.js
git commit -m "fixup! Add user authentication with JWT tokens"
```

## Command Reference

### Checking branch safety
```bash
# Current branch name
git branch --show-current

# List all authors on feature branch
git log origin/main..HEAD --format="%an" | sort -u

# Check if branch has merge commits
git log --oneline --merges HEAD ^origin/main
```

### Creating commits
```bash
# Regular commit
git commit -m "message"

# Amend last commit
git commit --amend --no-edit

# Fixup commit (autosquash)
git commit -m "fixup! target commit message"

# Interactive staging
git add -p
```

### Rebasing
```bash
# Find branch base (merge point with main)
git merge-base origin/main HEAD

# Interactive rebase from branch base
git rebase -i --autosquash $(git merge-base origin/main HEAD)

# Interactive rebase from specific commit (one before first to squash)
git rebase -i --autosquash <base-commit>^

# Interactive rebase last N commits
git rebase -i --autosquash HEAD~N

# Continue after resolving conflicts
git rebase --continue

# Abort rebase
git rebase --abort
```

### Branch synchronization
```bash
# Fetch latest changes from remote
git fetch origin

# Check if branch has merge commits
git log --oneline --merges HEAD ^origin/main

# If NO merge commits (output empty), sync with rebase
git rebase origin/main

# If HAS merge commits, sync with merge
git merge origin/main

# Find last merge commit (if any)
git log --oneline --merges HEAD ^origin/main | head -1
```

### Inspecting history
```bash
# View commit history
git log --oneline

# View commits not in main
git log --oneline origin/main..HEAD

# View when file was last changed
git log --oneline -- path/to/file

# View changes in a commit
git show <commit-hash>
```

## Decision Matrix

| Situation | Action | Command |
|-----------|--------|---------|
| New logical change | Regular commit | git commit -m "message" |
| Fix last commit | Amend | git commit --amend |
| Fix earlier commit | Fixup commit | git commit -m "fixup! original message" |
| PR comment (minor) | Fixup commit | git commit -m "fixup! original message" |
| PR comment (major) | Regular commit | git commit -m "descriptive message" |
| User asks to clean up fixups | Rebase with autosquash | git rebase -i --autosquash <base> |
| Sync with base (no merges) | Rebase | git rebase origin/main |
| Sync with base (has merges) | Merge | git merge origin/main |
| On protected branch | DO NOT use workflow | Regular commits only |
| Multiple authors | DO NOT use workflow | Regular commits only |
| Branch has merge commits | DO NOT rebase over merges | Use merge or rebase from after merge |

## Integration with Claude Code

When Claude Code is:
- Creating git commits to Apply atomic commit principles
- Fixing code issues to Determine if fixup or new commit is appropriate
- Asked to prepare for PR to Suggest running interactive rebase with autosquash (do not run automatically)
- Syncing branch to Check for merge commits first, then rebase or merge appropriately
- Asked to commit changes to Check branch safety first, then follow workflow

Always verify:
1. Current branch is not protected
2. Only one author on the branch
3. No merge commits exist (or rebase from after them)
4. Commit is cohesive and atomic
5. Message is clear and descriptive

IMPORTANT: Do not run interactive rebase automatically. Suggest it when appropriate, but let user decide when to execute.
