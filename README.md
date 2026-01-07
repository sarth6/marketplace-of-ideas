<!-- BANNER_IMAGE_PLACEHOLDER -->

# Marketplace of Ideas 💡

**Stop context-switching. Start shipping.**

This is a collection of Claude Code plugins that automate the tedious parts of open source contribution—so you can focus on writing great code.

---

## Why This Exists

Contributing to open source should be about solving interesting problems, not spending hours:
- Reading through massive GitHub issue threads
- Figuring out where code changes should go
- Understanding how a codebase is structured
- Writing design docs that will actually get approved

**These plugins do the grunt work for you.** They read the issues, explore the codebase, and produce artifacts that accelerate your contribution workflow.

---

## Quick Start

**1. Add the marketplace** (one-time setup):
```bash
/plugin marketplace add sarth6/marketplace-of-ideas
```

**2. Install a plugin**:
```bash
/plugin install open-source-contributor@marketplace-of-ideas
```

**3. Use it**:
```bash
/solve-issue https://github.com/some-org/some-repo/issues/123
```

That's it. You now have a comprehensive design doc in your working directory.

---

## Available Plugins

### 🔧 open-source-contributor

**AI-powered tools for contributing to open source projects.**

| Command | What it does |
|---------|--------------|
| `/solve-issue <url>` | Reads a GitHub issue, explores the codebase, and writes a design doc |
| `/solve-issue <url> --interactive` | Same thing, but pauses at checkpoints so you can guide the process |

**What happens when you run `/solve-issue`:**

1. **Fetches the issue** — Reads the full issue thread, extracts requirements, finds linked PRs
2. **Explores the codebase** — Deploys parallel agents to understand architecture, find related code, learn testing patterns
3. **Researches solutions** — Searches how other libraries solve similar problems
4. **Writes the design doc** — Produces a comprehensive, actionable document
5. **Self-reviews** — Checks for completeness and clarity before saving

**Output:** `design-doc-issue-{number}.md` in your current directory

---

## Example Workflow

```bash
# You find an interesting issue you want to tackle
/solve-issue https://github.com/pydantic/pydantic-ai/issues/1771

# 5 minutes later, you have a design doc that covers:
# - Problem statement and goals
# - Proposed solution with code examples
# - Implementation plan broken into phases
# - Testing strategy
# - Open questions for maintainers

# Now you can start coding with confidence, or open a discussion
# with the maintainers using your design doc as the basis
```

---

## Contributing

Want to add your own plugin? PRs welcome!

**Plugin structure:**
```
plugins/your-plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── commands/
│   └── your-command.md      # Slash commands
├── agents/                  # (optional) Custom agents
├── skills/                  # (optional) Skills
└── hooks/                   # (optional) Hooks
```

**Then add it to** `.claude-plugin/marketplace.json`:
```json
{
  "plugins": [
    {
      "name": "your-plugin-name",
      "source": "./plugins/your-plugin-name",
      "description": "What your plugin does"
    }
  ]
}
```

---

## License

MIT — use these plugins however you want.
