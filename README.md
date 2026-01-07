# Marketplace of Ideas 💡

A curated collection of Claude Code plugins for developers who want to get things done.

## Installation

Add this marketplace to Claude Code:

```bash
/plugin marketplace add sarth6/marketplace-of-ideas
```

Then install any plugin from it:

```bash
/plugin install open-source-contributor@marketplace-of-ideas
```

## Available Plugins

### open-source-contributor

Tools for contributing to open source projects with AI assistance.

**Commands:**

| Command | Description |
|---------|-------------|
| `/solve-issue <url>` | Analyzes a GitHub issue and produces a comprehensive design doc |
| `/solve-issue <url> --interactive` | Same as above, but pauses at checkpoints for your input |

**Example:**
```bash
/solve-issue https://github.com/pydantic/pydantic-ai/issues/1771
```

This will:
1. Fetch and analyze the GitHub issue
2. Deploy explore agents to understand the codebase
3. Research how other libraries solve similar problems
4. Write a comprehensive design document
5. Save it as `design-doc-issue-1771.md` in your current directory

## Contributing

Want to add a plugin? Open a PR! Each plugin should:

1. Live in `plugins/<plugin-name>/`
2. Have a `.claude-plugin/plugin.json` manifest
3. Include commands in a `commands/` directory
4. Follow Claude Code plugin conventions

Then add your plugin to `.claude-plugin/marketplace.json` at the repo root.

## License

MIT
