# Creating Claude Skills

A guide to creating custom skills for Claude Code.

## Skill Structure

A typical skill consists of:

```
my-skill/
├── skill.json          # Skill manifest
├── README.md           # Documentation
├── index.js            # Main implementation (or index.py, etc.)
└── tests/             # Optional: tests for your skill
```

## Skill Manifest (skill.json)

The `skill.json` file defines your skill's metadata and capabilities:

```json
{
  "name": "my-skill",
  "version": "1.0.0",
  "description": "What your skill does",
  "author": "Your Name",
  "commands": [
    {
      "name": "command-name",
      "description": "Command description",
      "parameters": [
        {
          "name": "param1",
          "type": "string",
          "required": true,
          "description": "Parameter description"
        }
      ]
    }
  ],
  "triggers": ["keyword1", "keyword2"]
}
```

## Implementation

Create your skill implementation in the language of your choice:

- JavaScript/TypeScript: `index.js` or `index.ts`
- Python: `index.py` or `__init__.py`
- Other languages as supported by Claude Code

## Best Practices

1. **Clear Documentation**: Provide clear README with examples
2. **Error Handling**: Handle errors gracefully
3. **Validation**: Validate inputs before processing
4. **Testing**: Include tests when appropriate
5. **Versioning**: Follow semantic versioning

## Testing Your Skill

Before publishing, test your skill:

1. Load it in Claude Code
2. Try various command combinations
3. Test edge cases and error conditions
4. Verify documentation accuracy

## Publishing

1. Ensure all documentation is complete
2. Tag releases with version numbers
3. Update changelog
4. Share with the community (if open source)
