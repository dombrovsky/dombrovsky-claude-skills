---
name: csharp-code-cleanup
description: Automated C# code cleanup using ReSharper CLI to fix StyleCop violations, format C# code, and maintain consistent code style in .NET projects. C# ONLY - not for other languages.
license: MIT
metadata:
  author: Volodymyr Dombrovskyi
  version: "1.0.0"
---

# C# Code Cleanup Skill

**⚠️ C# ONLY** - This skill is specifically for C# code formatting and StyleCop violations in .NET projects. It does NOT support other languages (Java, TypeScript, Python, etc.).

Automated code cleanup using ReSharper CLI (`jb cleanupcode`) to fix StyleCop violations, enforce code style standards, and maintain consistent formatting in C# projects.

## When to Use This Skill

### Manual Invocation
- User explicitly requests code cleanup: `/csharp-cleanup`, `/resharper-cleanup`, `/stylecop-fix`
- User asks to "fix StyleCop violations" or "format code"
- User wants to run cleanup before committing changes

### Assisted Detection (Automatic Suggestion)
- You detect SA#### violations in build output → suggest cleanup
- User's build fails with StyleCop errors → offer to fix automatically
- After modifying C# files → suggest cleanup before commit (optional)

**IMPORTANT:** Always require user confirmation before running cleanup. Never run automatically without permission.

## Core Workflow

### 1. Tool Installation Check

Before running cleanup, verify ReSharper CLI is installed:

```bash
jb cleanupcode --version
```

**If not installed:**
```bash
# Offer to install globally
dotnet tool install -g JetBrains.ReSharper.GlobalTools
```

**Verification after install:**
```bash
jb cleanupcode --version
# Should output version info (e.g., "JetBrains CleanupCode 2024.3.0")
```

### 2. Determine Cleanup Scope

**Default scope: Modified files only**
```bash
# Get list of modified .cs files
git diff --name-only --diff-filter=ACMR "*.cs"
git diff --cached --name-only --diff-filter=ACMR "*.cs"
```

**Alternative scopes:**
- Specific files: User-provided paths
- Entire solution: All .cs files in solution
- Pattern-based: Files matching glob pattern (e.g., `src/**/*.cs`)

### 3. Safety Checks

Before executing cleanup, verify:

```bash
# 1. Find solution file (.sln or .slnx)
# Supports both traditional .sln and XML-based .slnx formats
SOLUTION_FILE=$(find . -maxdepth 1 \( -name "*.sln" -o -name "*.slnx" \) | head -1)
if [[ -z "$SOLUTION_FILE" ]]; then
  echo "ERROR: No solution file (.sln or .slnx) found in current directory"
  exit 1
fi
echo "Found solution: $SOLUTION_FILE"

# 2. Check for .DotSettings file (optional but recommended)
SOLUTION_SETTINGS="${SOLUTION_FILE}.DotSettings"
if [[ -f "$SOLUTION_SETTINGS" ]]; then
  echo "Using solution settings: $SOLUTION_SETTINGS"
fi
```

**Note:** ReSharper CLI supports both `.sln` (traditional) and `.slnx` (XML-based Visual Studio solution format) files.

### 4. Run ReSharper Cleanup

**For specific files:**
```bash
jb cleanupcode MySolution.sln \
  --profile="Built-in: Full Cleanup" \
  --settings=MySolution.sln.DotSettings \
  --include="src/file1.cs;src/file2.cs;tests/file3.cs" \
  --verbosity=WARN
```

**For entire solution:**
```bash
jb cleanupcode MySolution.sln \
  --profile="Built-in: Full Cleanup" \
  --settings=MySolution.sln.DotSettings \
  --verbosity=WARN
```

**Command parameters:**
- `--profile`: Cleanup profile to use (default: "Built-in: Full Cleanup")
- `--settings`: Path to .DotSettings file with cleanup configuration
- `--include`: Semicolon-separated list of files to clean (optional)
- `--verbosity`: Output verbosity (VERBOSE, INFO, WARN, ERROR)

### 5. Parse Output and Report Results

ReSharper CLI output format:
```
JetBrains CleanupCode 2024.3.0
Running in 64-bit mode, .NET Core 6.0.36 under Microsoft Windows 10.0.26200
Processing solution file MySolution.sln
  Inspecting project MyApp.Api
  Inspecting project MyApp.Tests
Cleanup completed in 15.2s
```

**Report to user:**
```
✓ Code cleanup completed successfully
  - Cleaned up 5 files in 15.2s
  - Modified: src/Services/UserService.cs
  - Modified: src/Persistence/Repository.cs
  - Modified: tests/UserServiceTests.cs

Run 'dotnet build' to verify the cleanup fixed violations.
```

### 6. Verify Cleanup Success

Suggest rebuilding to verify violations are fixed:
```bash
dotnet build --no-restore
```

If violations persist, report them and suggest configuration alignment.

## StyleCop Violation Detection

When you see build output, scan for StyleCop violations:

**Pattern to detect:** `error SA####:` or `warning SA####:`

**Example build output:**
```
src/Services/UserService.cs(10,1): error SA1200: Using directive should appear within a namespace declaration [MyApp.Api.csproj]
src/Persistence/Repository.cs(25,9): error SA1309: Field '_repository' should not begin with an underscore [MyApp.Api.csproj]
```

**Detection logic:**
1. Parse lines containing `SA####`
2. Extract violation code (e.g., SA1200, SA1309)
3. Extract file path and line number
4. Group violations by file

**Offer cleanup:**
```
I detected StyleCop violations in the build output:
  - SA1200: Using directive placement (2 files)
  - SA1309: Field naming (1 file)

Would you like me to run ReSharper cleanup to fix these? (/csharp-cleanup)
```

## Command Variants

### `/csharp-cleanup` (default)
Runs cleanup on modified files only:
```bash
# Get modified files
MODIFIED_FILES=$(git diff --name-only --diff-filter=ACMR "*.cs" && git diff --cached --name-only --diff-filter=ACMR "*.cs")

# Run cleanup
jb cleanupcode MySolution.sln \
  --profile="Built-in: Full Cleanup" \
  --settings=MySolution.sln.DotSettings \
  --include="$MODIFIED_FILES"
```

### `/csharp-cleanup --all`
Runs cleanup on entire solution:
```bash
jb cleanupcode MySolution.sln \
  --profile="Built-in: Full Cleanup" \
  --settings=MySolution.sln.DotSettings
```

### `/csharp-cleanup --verify`
Dry-run mode (shows what would change without applying):
```bash
# ReSharper CLI doesn't have built-in dry-run
# Instead: create git stash, run cleanup, show diff, restore stash
git stash push -m "pre-cleanup-verify"
jb cleanupcode MySolution.sln --include="$FILES"
git diff --stat
echo "Run '/csharp-cleanup' to apply these changes"
git reset --hard HEAD
git stash pop
```

### `/csharp-cleanup file1.cs file2.cs`
Runs cleanup on specific files:
```bash
jb cleanupcode MySolution.sln \
  --profile="Built-in: Full Cleanup" \
  --settings=MySolution.sln.DotSettings \
  --include="file1.cs;file2.cs"
```

### `/csharp-cleanup src/**/*.cs`
Runs cleanup on files matching pattern:
```bash
# Expand glob pattern to file list
FILES=$(find src -name "*.cs" -type f | tr '\n' ';')
jb cleanupcode MySolution.sln \
  --profile="Built-in: Full Cleanup" \
  --settings=MySolution.sln.DotSettings \
  --include="$FILES"
```

### `/csharp-cleanup --profile="StyleCop Full Cleanup"`
Uses specific cleanup profile:
```bash
jb cleanupcode MySolution.sln \
  --profile="StyleCop Full Cleanup" \
  --settings=MySolution.sln.DotSettings
```

### `/csharp-cleanup --list-profiles`
Lists available cleanup profiles:
```bash
# ReSharper CLI doesn't have --list-profiles command
# Read from .DotSettings file or show built-in profiles
echo "Available cleanup profiles:"
echo "  - Built-in: Full Cleanup (default)"
echo "  - Built-in: Reformat Code"
echo "  - Built-in: Reformat & Apply Code Style"
echo ""
echo "Custom profiles (from .DotSettings):"
grep -A1 'CleanupProfile' MySolution.sln.DotSettings || echo "  (none found)"
```

## Safety Features

### Uncommitted Changes Warning
```bash
# Check if there are uncommitted changes
if [[ -n $(git status --porcelain) ]]; then
  echo "Warning: You have uncommitted changes. Cleanup will modify files."
  echo "Consider committing or stashing changes first."
  # Require explicit confirmation
fi
```

### File Preview
Before running cleanup on many files, show preview:
```bash
FILE_COUNT=$(echo "$MODIFIED_FILES" | wc -l)
if [[ $FILE_COUNT -gt 10 ]]; then
  echo "Will clean up $FILE_COUNT files:"
  echo "$MODIFIED_FILES" | head -10
  echo "...and $(($FILE_COUNT - 10)) more"
  echo ""
  echo "Proceed with cleanup? (y/n)"
fi
```

## Error Handling

### ReSharper CLI Not Installed
```
ReSharper CLI (jb cleanupcode) is not installed.

Install globally with:
  dotnet tool install -g JetBrains.ReSharper.GlobalTools

Or install locally:
  dotnet tool install JetBrains.ReSharper.GlobalTools

Would you like me to install it globally? (y/n)
```

### Solution File Not Found
```
ERROR: No solution file (.sln or .slnx) found in current directory.

Please navigate to the solution directory or ensure a solution file exists.
Supported formats:
  - .sln (Visual Studio solution)
  - .slnx (XML-based Visual Studio solution)
```

### Cleanup Failed
```
ERROR: ReSharper cleanup failed with exit code 1

Common causes:
  - Solution cannot be restored (missing NuGet packages)
  - Compilation errors in the solution
  - Invalid .DotSettings configuration

Try running 'dotnet build' first to identify issues.
```

### Violations Persist After Cleanup
```
Warning: StyleCop violations still present after cleanup:
  - SA1200: Using directive placement (2 files)

This may indicate:
  1. .DotSettings and .editorconfig have conflicting rules
  2. Some rules require manual fixes (e.g., SA1600 documentation)
  3. Cleanup profile doesn't include these rules

Would you like me to check configuration alignment?
```

## Fallback: dotnet format

If ReSharper CLI is not available and user declines installation:

```bash
# Use dotnet format as fallback
dotnet format MySolution.sln --verify-no-changes --verbosity diagnostic

# If changes needed:
dotnet format MySolution.sln
```

**Warning message:**
```
Note: Using 'dotnet format' instead of ReSharper CLI.
  - Only .editorconfig rules will be applied
  - .DotSettings cleanup profiles will be ignored
  - Some StyleCop rules may not be fixed

For full cleanup support, install ReSharper CLI.
```

## Integration with Other Skills

### Git Atomic Workflow Integration
When user is about to commit, suggest cleanup:
```
You have modified 3 C# files with unstaged changes.
Would you like to run code cleanup before committing? (/csharp-cleanup)
```

### Build Integration
After detecting build failures with SA#### violations:
```
Build failed with 5 StyleCop violations.
Run /csharp-cleanup to automatically fix these violations.
```

## Configuration Files Reference

### .DotSettings Files (ReSharper Configuration)
- `<SolutionName>.sln.DotSettings` - Solution-level settings
- `CompanyWide.DotSettings` - Company-wide standards (optional)
- Project-specific `.DotSettings` files

**Used by:** ReSharper CLI (`jb cleanupcode`)
**Contains:** Cleanup profiles, code style settings, formatting rules

### .editorconfig (EditorConfig Standard)
- `.editorconfig` - Typically in solution root

**Used by:** IDEs, dotnet format, analyzers
**Contains:** Formatting rules, analyzer severity, code style preferences

### stylecop.json (StyleCop Analyzer Configuration)
- `stylecop.json` - Typically in solution root

**Used by:** StyleCop.Analyzers NuGet package
**Contains:** StyleCop-specific settings (documentation rules, naming rules)

## Limitations

### What ReSharper Cleanup Cannot Fix
- **SA1600-SA1649**: Documentation rules (require manual documentation)
- **SA1300-SA1316**: Naming rules that require renaming (may break API compatibility)
- **SA0001**: XML comment analysis (disabled in many projects)
- Complex refactoring that requires understanding business logic

### Cleanup Can Introduce Changes That:
- Affect git history (formatting changes in unrelated lines)
- Trigger PR review for style changes
- May conflict with other branches

**Recommendation:** Run cleanup on isolated changes, not on large feature branches with many modifications.

### Performance
- Full solution cleanup can take 30-60 seconds on large codebases
- Modified file cleanup is faster (5-10 seconds for 5-10 files)
- ReSharper CLI requires solution restore (downloads NuGet packages if needed)

## Checklist Before Running Cleanup

- [ ] ReSharper CLI is installed (`jb cleanupcode --version`)
- [ ] Solution file (.sln or .slnx) exists and is valid
- [ ] .DotSettings configuration file exists (optional but recommended)
- [ ] User has confirmed they want to run cleanup
- [ ] Scope is clear (modified files, all files, specific files)
- [ ] User is aware cleanup will modify files

## Success Indicators

- ✅ Cleanup completes without errors
- ✅ Build succeeds after cleanup (`dotnet build`)
- ✅ StyleCop violations are resolved
- ✅ Git diff shows only formatting/style changes
- ✅ Code still compiles and tests pass
