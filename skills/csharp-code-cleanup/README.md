# C# Code Cleanup Skill

**⚠️ C# ONLY** - This skill is specifically for C# code formatting in .NET projects. It does NOT support other languages.

Automated code cleanup for C# projects using ReSharper CLI to fix StyleCop violations and maintain consistent code style.

## Quick Start

### Installation

The skill uses ReSharper CLI (`jb cleanupcode`), which will be automatically installed when first used:

```bash
dotnet tool install -g JetBrains.ReSharper.GlobalTools
```

### Basic Usage

```bash
# Clean up modified files
/csharp-cleanup

# Clean up entire solution
/csharp-cleanup --all

# Clean up specific files
/csharp-cleanup src/Services/MyService.cs tests/MyTests.cs

# Preview changes without applying
/csharp-cleanup --verify

# Use specific cleanup profile
/csharp-cleanup --profile="Built-in: Reformat Code"
```

## When to Use

### Automatic Detection

Claude will automatically detect StyleCop violations in build output and offer to fix them:

```
Build output shows:
  error SA1200: Using directive should appear within namespace declaration
  error SA1309: Field should not begin with underscore

Claude: "I detected StyleCop violations. Run /csharp-cleanup to fix them?"
```

### Manual Invocation

Run cleanup explicitly when you want to:
- Fix code style violations before committing
- Format code after making changes
- Ensure consistent style across files
- Prepare code for PR review

### Aliases

All commands do the same thing:
- `/csharp-cleanup`
- `/resharper-cleanup`
- `/stylecop-fix`

## How It Works

1. **Checks** if ReSharper CLI is installed (offers to install if not)
2. **Determines** which files to clean (modified files by default)
3. **Verifies** solution file (.sln or .slnx) exists
4. **Runs** `jb cleanupcode` with project's `.DotSettings` configuration
5. **Reports** results and suggests rebuilding to verify

## Configuration

The skill uses your project's existing configuration:
- **`.DotSettings`** files - ReSharper cleanup profiles and code style
- **`.editorconfig`** - EditorConfig formatting rules
- **`stylecop.json`** - StyleCop analyzer settings

No additional configuration needed - it respects your project's standards.

## Safety Features

### File Preview
Shows which files will be modified before running cleanup (when > 10 files).

### Explicit Confirmation
Always asks for confirmation before modifying files.

### Verification Suggestion
Recommends running `dotnet build` after cleanup to verify fixes.

## Command Reference

### Scope Options

| Command | Description | Files Affected |
|---------|-------------|----------------|
| `/csharp-cleanup` | Default | Modified files only (git diff) |
| `/csharp-cleanup --all` | Full solution | All .cs files in solution |
| `/csharp-cleanup file1.cs file2.cs` | Specific files | Only specified files |
| `/csharp-cleanup src/**/*.cs` | Pattern | Files matching glob pattern |

### Special Options

| Command | Description |
|---------|-------------|
| `/csharp-cleanup --verify` | Dry-run (preview changes) |
| `/csharp-cleanup --profile="Name"` | Use specific cleanup profile |
| `/csharp-cleanup --list-profiles` | Show available profiles |

## Examples

### Fix Build Violations

```
User: dotnet build

Build output:
  error SA1200: Using directive should appear within namespace declaration

Claude: "Detected SA1200 violation. Run /csharp-cleanup?"
User: yes

Claude runs cleanup and reports:
  ✓ Cleaned up 3 files in 8.5s
  Run 'dotnet build' to verify the fix.
```

### Pre-Commit Cleanup

```
User: I've modified several files, let's clean them up before committing

Claude: /csharp-cleanup

Will clean up 5 files:
  - src/Services/UserSchedulingService.cs
  - src/Persistence/MongoDbRepository.cs
  - tests/UserSchedulingServiceTests.cs
  - tests/MongoDbRepositoryTests.cs
  - tests/Integration/EndToEndTests.cs

Proceed? (y/n)

User: y

Claude: ✓ Cleanup completed successfully
```

### Preview Changes

```
User: /csharp-cleanup --verify

Claude:
Running cleanup in dry-run mode...

Would modify 3 files:
  src/Services/UserSchedulingService.cs (42 lines changed)
  src/Persistence/MongoDbRepository.cs (15 lines changed)
  tests/UserSchedulingServiceTests.cs (8 lines changed)

Run '/csharp-cleanup' to apply these changes.
```

## Troubleshooting

### ReSharper CLI Not Installed

**Symptom:** `jb: command not found`

**Solution:**
```bash
dotnet tool install -g JetBrains.ReSharper.GlobalTools
```

Or let Claude install it when prompted.

### Solution Cannot Be Restored

**Symptom:** `ERROR: Failed to restore solution`

**Solution:**
```bash
dotnet restore
```

### Violations Persist After Cleanup

**Symptom:** Build still shows SA#### errors after cleanup

**Possible causes:**
1. **Rule requires manual fix** - Some rules like SA1600 (documentation) cannot be automated
2. **Configuration conflict** - `.DotSettings` and `.editorconfig` have different rules
3. **Profile doesn't include rule** - Cleanup profile may not cover all StyleCop rules

**Solution:**
- Check which violations persist: `dotnet build | grep "error SA"`
- For SA1600 (documentation), add XML comments manually
- For configuration conflicts, ask Claude to check alignment

### Cleanup Changes Too Many Files

**Symptom:** Cleanup modifies files you didn't change

**Cause:** Running `/csharp-cleanup --all` on solution with existing style violations

**Solution:**
- Use `/csharp-cleanup` (no arguments) to clean only modified files
- Or clean specific files: `/csharp-cleanup path/to/file.cs`

## Fallback: dotnet format

If ReSharper CLI is not available, the skill can fall back to `dotnet format`:

```bash
dotnet format MySolution.sln
```

**Limitations of dotnet format:**
- Only uses `.editorconfig` rules
- Ignores `.DotSettings` cleanup profiles
- May not fix all StyleCop violations

**Recommendation:** Install ReSharper CLI for full cleanup support.

## Integration with Git Workflow

Works seamlessly with git-atomic-workflow skill:

```
User: I'm ready to commit my changes

Claude: You have 3 modified C# files. Run /csharp-cleanup first?
User: yes

Claude runs cleanup, then proceeds with commit workflow
```

## What Gets Fixed

### Formatting
- Brace placement
- Indentation
- Line spacing
- Line breaks

### Code Style
- Using directive placement (inside/outside namespace)
- Field naming (`_camelCase` for private fields)
- Modifier order (`public static async Task`)
- Expression preferences (pattern matching, null checks)

### StyleCop Rules
- **SA1200**: Using directives placement
- **SA1309**: Field naming
- **SA1028**: Code must not contain trailing whitespace
- **SA1005**: Single line comment spacing
- **SA1400-SA1413**: Access modifier rules
- Many more...

### What Doesn't Get Fixed
- **Documentation** (SA1600-SA1649) - Requires manual XML comments
- **Naming that breaks APIs** - Won't rename public members
- **Complex refactoring** - Logic-level changes

## Performance

| Scope | Typical Time | Files |
|-------|--------------|-------|
| Modified files (1-5) | 5-10 seconds | 1-5 |
| Modified files (5-20) | 10-20 seconds | 5-20 |
| Entire solution | 30-60 seconds | 100+ |

**First run** may take longer due to NuGet package restore.

## Tips

1. **Run cleanup before commits** - Keeps git history clean
2. **Use modified files scope** - Faster and more focused
3. **Preview large changes** - Use `--verify` for big cleanups
4. **Check build after cleanup** - Verify violations are fixed
5. **Commit cleanup separately** - Makes PR review easier

## Requirements

- .NET SDK 6.0 or later
- Git repository (for detecting modified files)
- Solution file (`.sln` or `.slnx`)
- ReSharper CLI (installed automatically when needed)

## License

MIT License - Free to use and modify

## Author

Volodymyr Dombrovskyi

## Version

1.0.0
