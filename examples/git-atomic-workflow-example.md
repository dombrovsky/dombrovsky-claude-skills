# Git Atomic Workflow - Example Usage

This example demonstrates the git-atomic-workflow skill in action.

## Scenario: Building a User Authentication Feature

### Initial Development

You're building a user authentication feature. Here's how the workflow looks:

#### Step 1: Create Feature Branch
```bash
$ git checkout -b feature/user-authentication
Switched to a new branch 'feature/user-authentication'
```

#### Step 2: First Logical Commit
```bash
$ git add src/auth/login.js src/auth/types.ts
$ git commit -m "Add login form component with validation"
[feature/user-authentication abc1234] Add login form component with validation
 2 files changed, 145 insertions(+)
```

#### Step 3: Second Logical Commit
```bash
$ git add src/auth/api.js
$ git commit -m "Implement JWT authentication API"
[feature/user-authentication def5678] Implement JWT authentication API
 1 file changed, 87 insertions(+)
```

#### Step 4: Third Logical Commit
```bash
$ git add src/auth/middleware.js src/routes/protected.js
$ git commit -m "Add authentication middleware for protected routes"
[feature/user-authentication ghi9012] Add authentication middleware for protected routes
 2 files changed, 64 insertions(+)
```

### Fixing Earlier Work

#### Step 5: Realize Login Form Needs Fix
You notice the login form is missing error handling. This logically belongs with the first commit:

```bash
$ git add src/auth/login.js
$ git commit -m "fixup! Add login form component with validation"
[feature/user-authentication jkl3456] fixup! Add login form component with validation
 1 file changed, 12 insertions(+)
```

#### Step 6: Continue Development
```bash
$ git add src/auth/logout.js
$ git commit -m "Add logout functionality"
[feature/user-authentication mno7890] Add logout functionality
 1 file changed, 34 insertions(+)
```

#### Step 7: API Needs Improvement
The JWT API needs better token refresh logic:

```bash
$ git add src/auth/api.js
$ git commit -m "fixup! Implement JWT authentication API"
[feature/user-authentication pqr1234] fixup! Implement JWT authentication API
 1 file changed, 28 insertions(+)
```

### Current State
```bash
$ git log --oneline origin/main..HEAD
pqr1234 fixup! Implement JWT authentication API
mno7890 Add logout functionality
jkl3456 fixup! Add login form component with validation
ghi9012 Add authentication middleware for protected routes
def5678 Implement JWT authentication API
abc1234 Add login form component with validation
```

### PR Review Process

#### Step 8: PR Reviewer Comments
Reviewer says: "The middleware should also log authentication attempts"

This is substantial enough for its own commit:

```bash
$ git add src/auth/middleware.js
$ git commit -m "Add authentication attempt logging to middleware"
[feature/user-authentication stu5678] Add authentication attempt logging to middleware
 1 file changed, 15 insertions(+)
```

#### Step 9: Another PR Comment
Reviewer says: "Fix typo in logout button text"

This is minor, use fixup:

```bash
$ git add src/auth/logout.js
$ git commit -m "fixup! Add logout functionality"
[feature/user-authentication vwx9012] fixup! Add logout functionality
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### Cleaning Up History

#### Step 10: Interactive Rebase with Autosquash
```bash
$ git rebase -i --autosquash origin/main
```

The rebase automatically:
1. Moves "fixup! Add login form component" right after "Add login form component"
2. Moves "fixup! Implement JWT authentication API" right after "Implement JWT authentication API"
3. Moves "fixup! Add logout functionality" right after "Add logout functionality"
4. Squashes all fixup commits into their targets

### Final Clean History
```bash
$ git log --oneline origin/main..HEAD
xyz3456 Add authentication attempt logging to middleware
mno7890 Add logout functionality
ghi9012 Add authentication middleware for protected routes
def5678 Implement JWT authentication API
abc1234 Add login form component with validation
```

Perfect! Five meaningful commits that tell the story of the feature development.

## Advanced Example: Partial File Staging

### Scenario: Multiple Fixes in One File

You're working on `src/dashboard/widgets.js` and realize it needs fixes that belong to two different previous commits:

```bash
$ git log --oneline
abc1234 Add weather widget
def5678 Add news widget
ghi9012 Add layout grid
```

Your working changes include:
- Weather widget bug fix
- News widget styling improvement

#### Solution: Interactive Staging

```bash
# Stage only the weather widget changes
$ git add -p src/dashboard/widgets.js
# Select hunks related to weather widget (y)
# Reject hunks related to news widget (n)

$ git commit -m "fixup! Add weather widget"

# Stage the remaining news widget changes
$ git add -p src/dashboard/widgets.js
# Select remaining hunks (y)

$ git commit -m "fixup! Add news widget"
```

Now both fixes will be squashed into their correct logical commits during rebase.

## Safety Check Example

### Scenario: Multiple Authors

```bash
$ git log origin/main..HEAD --format="%an" | sort -u
Alice Johnson
Bob Smith
```

**⚠️ STOP! Do not rebase this branch.**

Multiple authors have contributed. Rebasing would rewrite their commit history. Use regular commits only.

### Scenario: Protected Branch

```bash
$ git branch --show-current
main
```

**⚠️ STOP! Do not rebase protected branches.**

This is a protected branch. Use regular commit workflow only.

## Conflict Avoidance Example

### Scenario: Potential Conflict

```bash
# Recent commits
abc1234 Refactor database connection (changed: src/db/connection.js)
def5678 Add user queries (changed: src/db/queries.js)
ghi9012 Add caching layer (changed: src/db/connection.js, src/db/cache.js)
```

You need to fix a bug in `src/db/connection.js`.

**Problem**: If you create `fixup! Refactor database connection`, during rebase it will move before "Add caching layer", which also modified the same file. This could cause conflicts.

**Solution**: Attach to the most recent commit that touched the file:

```bash
$ git log --oneline -- src/db/connection.js
ghi9012 Add caching layer
abc1234 Refactor database connection

$ git commit -m "fixup! Add caching layer"
```

This avoids conflicts since the fixup won't be reordered past other changes to the same file.

## Branch Synchronization Example

### Scenario: Keeping Feature Branch Up to Date

You're working on a long-running feature branch and need to sync with main periodically.

#### Week 1: Initial Work (Clean, Linear History)

```bash
# Create feature branch
$ git checkout -b feature/payment-system
Switched to a new branch 'feature/payment-system'

# Do some work
$ git commit -m "Add payment processing core"
$ git commit -m "Implement credit card validation"
$ git commit -m "Add payment webhooks"

# Check current state
$ git log --oneline origin/main..HEAD
ghi9012 Add payment webhooks
def5678 Implement credit card validation
abc1234 Add payment processing core
```

#### Week 2: Sync with Main (No Merges Yet - Use Rebase)

```bash
# Fetch latest changes
$ git fetch origin

# Check if branch has any merge commits
$ git log --oneline --merges HEAD ^origin/main
# (empty output - no merge commits)

# Safe to rebase! Keeps linear history
$ git rebase origin/main
Successfully rebased and updated refs/heads/feature/payment-system.

# History is still clean and linear
$ git log --oneline origin/main..HEAD
ghi9012 Add payment webhooks
def5678 Implement credit card validation
abc1234 Add payment processing core
```

#### Week 3: More Work + Sync Again (Still Linear)

```bash
# More commits
$ git commit -m "Add PayPal integration"
$ git commit -m "Add refund processing"

# Fetch and check for merges
$ git fetch origin
$ git log --oneline --merges HEAD ^origin/main
# (still empty - no merges)

# Rebase again to stay current
$ git rebase origin/main
Successfully rebased and updated refs/heads/feature/payment-system.

# Still clean linear history
$ git log --oneline origin/main..HEAD
pqr1234 Add refund processing
mno7890 Add PayPal integration
ghi9012 Add payment webhooks
def5678 Implement credit card validation
abc1234 Add payment processing core
```

#### Week 4: Accidental Merge (Introduces Merge Commit)

```bash
# Oops! Someone did a merge instead of rebase
$ git merge origin/main
Merge made by the 'recursive' strategy.
# Now branch has a merge commit

# Check for merge commits
$ git log --oneline --merges HEAD ^origin/main
stu5678 Merge remote-tracking branch 'origin/main' into feature/payment-system
# ⚠️ Warning: Branch now has merge commit!

# View history (now has merge noise)
$ git log --oneline origin/main..HEAD
vwx9012 (more work after merge)
stu5678 Merge remote-tracking branch 'origin/main'
pqr1234 Add refund processing
mno7890 Add PayPal integration
ghi9012 Add payment webhooks
def5678 Implement credit card validation
abc1234 Add payment processing core
```

#### Week 5: Sync Again (Has Merge - Must Use Merge Strategy)

```bash
# Need to sync with main again
$ git fetch origin

# Check for merge commits
$ git log --oneline --merges HEAD ^origin/main
stu5678 Merge remote-tracking branch 'origin/main'
# ⚠️ Has merge commit - cannot safely rebase entire branch

# Option 1: Continue with merge strategy (consistent approach)
$ git merge origin/main
Merge made by the 'recursive' strategy.

# Option 2: Rebase only commits AFTER the last merge
$ git log --oneline --merges HEAD ^origin/main | head -1
stu5678 Merge remote-tracking branch 'origin/main'

# Rebase from after the merge
$ git rebase --onto origin/main stu5678 HEAD
Successfully rebased and updated refs/heads/feature/payment-system.
# This rebases only vwx9012 and later commits
```

### Best Practice Workflow

**Starting a new feature branch:**
```bash
# Always branch from latest main
git checkout main
git pull origin main
git checkout -b feature/new-feature

# Do work and commit
git commit -m "Add feature part 1"
git commit -m "Add feature part 2"
```

**Syncing during development (recommended):**
```bash
# Fetch latest
git fetch origin

# Check for merge commits
MERGES=$(git log --oneline --merges HEAD ^origin/main)

if [ -z "$MERGES" ]; then
  echo "No merge commits - safe to rebase"
  git rebase origin/main
else
  echo "⚠️ Branch has merge commits!"
  echo "Use: git merge origin/main"
  echo "OR rebase from after last merge"
fi
```

**What NOT to do:**
```bash
# ❌ DON'T: Blindly merge when rebase would work
git merge origin/main  # Creates unnecessary merge commits

# ❌ DON'T: Rebase when branch has merges
git rebase origin/main  # Will cause conflicts and duplicate commits

# ❌ DON'T: Mix strategies inconsistently
# Leads to confusing history
```

**Recovery from accidental merge:**
```bash
# If you just merged and want to undo
git reset --hard HEAD~1  # Removes the merge commit
git rebase origin/main   # Rebase instead

# Only do this if you haven't pushed!
```

## Integration with Claude Code

When using this skill with Claude Code:

### Example 1: Creating a Commit
**User**: "Commit the authentication changes"

**Claude Code** will:
1. Check if branch is safe (not protected, single author)
2. Review staged changes for cohesiveness
3. Create atomic commit with clear message
4. Suggest fixup if changes relate to previous commit

### Example 2: Addressing PR Comments
**User**: "Fix the PR comments - reviewer wants better error messages and a typo fixed"

**Claude Code** will:
1. Identify that typo fix should be a fixup
2. Identify that error message improvements might be fixup or new commit (depending on scope)
3. Create appropriate commits
4. Suggest running rebase when ready

### Example 3: Preparing for Review
**User**: "Clean up the branch for PR"

**Claude Code** will:
1. Verify branch safety
2. Run `git rebase -i --autosquash origin/main`
3. Resolve any conflicts if needed
4. Confirm clean history

## Summary

The git-atomic-workflow skill helps maintain professional, reviewable git history by:

✅ Encouraging atomic, logical commits
✅ Using fixup commits for minor fixes
✅ Automating history cleanup with autosquash
✅ Enforcing safety checks
✅ Preserving meaningful development context

This creates a git history that is both complete (for future reference) and clean (for review and understanding).
