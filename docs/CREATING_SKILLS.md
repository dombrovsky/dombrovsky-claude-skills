# Creating Agent Skills

A guide to creating custom skills following the [Agent Skills specification](https://agentskills.io/specification).

## Quick Start

A minimal skill requires only one file:

```
my-skill/
└── SKILL.md          # Required: Instructions with YAML frontmatter
```

## SKILL.md Format

Every skill must have a `SKILL.md` file with YAML frontmatter followed by markdown instructions.

### Minimal Example

```markdown
---
name: my-skill
description: Brief description of what this skill does and when to use it.
---

# My Skill

Instructions for Claude on how to use this skill...
```

### Full Example with Optional Fields

```markdown
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
license: MIT
compatibility: Requires Python 3.8+ with pypdf library
metadata:
  author: Your Name
  version: "1.0.0"
---

# PDF Processing Skill

## Overview
...
```

## YAML Frontmatter Fields

### Required Fields

**name** (required)
- Must be 1-64 characters
- Lowercase letters, numbers, and hyphens only
- Cannot start or end with hyphen
- Must match the parent directory name

**description** (required)
- Must be 1-1024 characters
- Should describe **what** the skill does **and when** to use it
- Include keywords that help agents identify relevant tasks

### Optional Fields

**license** (optional)
- License name or reference to bundled LICENSE file
- Example: MIT, Apache-2.0, or "Proprietary. See LICENSE.txt"

**compatibility** (optional)
- Max 500 characters
- Indicates environment requirements (system packages, network access, etc.)
- Example: "Requires git, docker, and internet access"

**metadata** (optional)
- Arbitrary key-value pairs for additional metadata

**allowed-tools** (optional, experimental)
- Space-delimited list of pre-approved tools
- Example: "Bash(git:*) Bash(jq:*) Read"

## Markdown Body

The content after the frontmatter contains instructions for Claude. There are no format restrictions - write whatever helps the agent perform the task effectively.

### Recommended Sections

- **Overview**: Brief explanation of the skill's purpose
- **Quick Start**: Common usage examples
- **Detailed Workflows**: Step-by-step instructions
- **Command Reference**: Commands and their usage
- **Examples**: Input/output examples
- **Common Issues**: Edge cases and troubleshooting

### Keep It Concise

- Aim for under 500 lines (~5000 tokens)
- Move detailed reference material to separate files
- Use progressive disclosure (see below)

## Optional Directories

### scripts/

Executable code that agents can run.

Scripts should be self-contained, include helpful error messages, and handle edge cases gracefully.

### references/

Additional documentation loaded on demand. Keep reference files focused and under 500 lines each.

### assets/

Static resources like templates, diagrams, and data files.

## Progressive Disclosure

Skills are loaded in stages to optimize context usage:

1. **Metadata** (~100 tokens): name and description loaded at startup
2. **Instructions** (<5000 tokens): Full SKILL.md loaded when activated
3. **Resources** (as needed): Reference files and scripts loaded on demand

Keep SKILL.md concise, move detailed documentation to references/, reference external files only when needed.

## Examples

### Example 1: Simple Prompt-Only Skill

```
git-atomic-workflow/
├── SKILL.md
└── README.md
```

### Example 2: Skill with Scripts and References

```
pdf-processing/
├── SKILL.md
├── README.md
├── scripts/
│   ├── extract.py
│   └── merge.py
└── references/
    ├── REFERENCE.md
    └── FORMS.md
```

## Validation

Validate your skill structure:

```bash
# Install skills-ref validator
npm install -g @agentskills/skills-ref

# Validate skill
skills-ref validate ./my-skill
```

## Best Practices

1. **Clear, Searchable Descriptions** - Include specific keywords and use cases
2. **Concise Instructions** - Keep SKILL.md under 500 lines
3. **Self-Contained Examples** - Include copy-pasteable code examples
4. **Error Handling** - Document common errors and troubleshooting
5. **Version Management** - Use semantic versioning

## Publishing

### For Public Skills

Add `.claude-plugin/marketplace.json` to your repository. Users can install via:
```bash
claude skills install github:user/repo
```

### For Private/Personal Skills

Copy skill directory to Claude Code skills location:
```bash
cp -r my-skill ~/.claude-code/skills/
```

## Resources

- **Official Specification**: https://agentskills.io/specification
- **Complete Documentation**: https://agentskills.io/llms.txt
- **Validator Tool**: https://github.com/agentskills/agentskills/tree/main/skills-ref
- **Example Skills**: https://github.com/anthropics/skills

## Common Issues

**Q: Should I use skill.json or SKILL.md?**
A: Use SKILL.md with YAML frontmatter (official spec). skill.json may be used by specific tools but is not part of the core specification.

**Q: How do I specify when my skill should activate?**
A: Use a clear, keyword-rich description field. The description is how agents discover skills.

**Q: Can I have multiple skills in one repository?**
A: Yes! Use .claude-plugin/marketplace.json to group skills into plugins.

**Q: How long should SKILL.md be?**
A: Aim for under 500 lines (~5000 tokens). Move detailed documentation to reference files.
