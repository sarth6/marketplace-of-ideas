# Contributing to Marketplace of Ideas

Thanks for your interest in contributing! This marketplace thrives on community plugins.

## Ways to Contribute

### 1. Add a New Plugin

The best way to contribute is by adding a plugin that solves a real problem you've faced.

**Plugin ideas we'd love to see:**
- Code review automation
- Documentation generators
- Testing helpers
- Deployment workflows
- Project scaffolding tools

### 2. Improve Existing Plugins

Found a bug or have an enhancement idea? PRs welcome!

### 3. Report Issues

If something isn't working, [open an issue](https://github.com/sarth6/marketplace-of-ideas/issues).

---

## Adding a New Plugin

### Step 1: Create the Plugin Structure

```
plugins/your-plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Required: Plugin manifest
├── commands/
│   └── your-command.md      # Slash commands (at least one)
├── agents/                  # Optional: Custom agents
├── skills/                  # Optional: Skills
└── hooks/                   # Optional: Hooks
```

### Step 2: Write Your plugin.json

```json
{
  "$schema": "https://claude.ai/code/plugin-schema.json",
  "name": "your-plugin-name",
  "version": "1.0.0",
  "description": "Clear, concise description of what your plugin does",
  "author": "your-github-username",
  "commands": "${CLAUDE_PLUGIN_ROOT}/commands"
}
```

### Step 3: Create Your Commands

Commands are markdown files with YAML frontmatter:

```markdown
---
description: What this command does
arguments:
  - name: arg_name
    description: What this argument is for
    required: true
---

# Your Command

Instructions for Claude Code to follow when this command is invoked.

Use $ARGUMENTS.arg_name to reference arguments.
```

### Step 4: Register in marketplace.json

Add your plugin to `.claude-plugin/marketplace.json`:

```json
{
  "plugins": [
    // ... existing plugins ...
    {
      "name": "your-plugin-name",
      "source": "./plugins/your-plugin-name",
      "description": "What your plugin does"
    }
  ]
}
```

### Step 5: Update the README

Add your plugin to the "Available Plugins" section in README.md.

### Step 6: Open a PR

- Use a clear PR title: `Add your-plugin-name plugin`
- Describe what problem your plugin solves
- Include example usage

---

## Code Style Guidelines

- **Plugin names**: Use `kebab-case` (e.g., `code-reviewer`, `doc-generator`)
- **Command names**: Use `kebab-case` (e.g., `solve-issue`, `review-pr`)
- **Descriptions**: Be clear and concise; start with a verb (e.g., "Analyzes...", "Generates...")
- **Commands**: Write clear, detailed instructions for Claude Code to follow

---

## Testing Your Plugin

Before submitting, test your plugin locally:

1. Install from your local path:
   ```bash
   /plugin install /path/to/marketplace-of-ideas/plugins/your-plugin-name
   ```

2. Run your commands and verify they work as expected

3. Test edge cases (invalid inputs, missing files, etc.)

---

## Questions?

[Open an issue](https://github.com/sarth6/marketplace-of-ideas/issues) and we'll help you out!
