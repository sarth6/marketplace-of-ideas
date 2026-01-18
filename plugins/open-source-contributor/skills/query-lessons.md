---
description: Query lessons learned from PR reviews for this repository. Use this skill before writing code to apply project conventions and avoid common review feedback. Supports filtering by category (code-style, architecture, testing, documentation, performance, security, conventions) or free-text search.
---

# Query Review Lessons

This skill provides programmatic access to lessons learned from PR review feedback. Use it to retrieve project-specific conventions and patterns before writing code.

## When to Use This Skill

- Before implementing a feature, query relevant lessons to apply project conventions
- When writing code in a specific category (testing, error handling, etc.)
- To check if there are known patterns for a particular type of code
- Before submitting code for review, to proactively address common feedback

## How to Query

### Step 1: Locate the Lessons Directory

```bash
# Get repo identifier from origin URL
REPO_ID=$(git remote get-url origin 2>/dev/null | shasum -a 256 | cut -c1-12)

# Fallback to repo path if no remote
if [ -z "$REPO_ID" ]; then
  REPO_ID=$(git rev-parse --show-toplevel | shasum -a 256 | cut -c1-12)
fi

LESSONS_DIR="$HOME/.claude/projects/$REPO_ID"
```

### Step 2: Read Available Lessons

```bash
# Read shared lessons (project conventions)
SHARED_FILE="$LESSONS_DIR/review-lessons.md"
if [ -f "$SHARED_FILE" ]; then
  cat "$SHARED_FILE"
fi

# Optionally read personal lessons
LOCAL_FILE="$LESSONS_DIR/review-lessons.local.md"
if [ -f "$LOCAL_FILE" ]; then
  cat "$LOCAL_FILE"
fi
```

### Step 3: Filter by Relevance

When querying for a specific task, filter lessons by:

**By Category:**
- `code-style` - Formatting, naming conventions, code organization
- `architecture` - Design patterns, module structure, dependencies
- `testing` - Test patterns, coverage requirements, mocking approaches
- `documentation` - Comment styles, docstring formats, README conventions
- `performance` - Optimization patterns, caching strategies
- `security` - Input validation, authentication patterns, data handling
- `conventions` - Project-specific rules and standards

**By Tags:**
Search for lessons with matching tags (e.g., "error-handling", "async", "validation")

**By Confidence:**
- `high` - Established project conventions, apply consistently
- `medium` - Strong preferences, apply when appropriate
- `low` - Single instances, consider but verify

## Response Format

Return relevant lessons in a structured format:

```markdown
## Relevant Lessons for [context]

### High Confidence (Always Apply)

1. **[Lesson title]**
   - Category: [category]
   - Applies to: [what code this affects]
   - Action: [what to do]

### Medium Confidence (Recommended)

2. **[Lesson title]**
   - Category: [category]
   - Note: [any caveats]

### Low Confidence (Consider)

3. **[Lesson title]**
   - Source: [single PR reference]
   - Note: May be personal preference, verify if uncertain
```

## Example Usage

### Example 1: Before Writing Tests

Query: "testing"

```markdown
## Relevant Lessons for Testing

### High Confidence
1. **Use table-driven tests for multiple cases**
   - Category: testing
   - Action: Structure tests as arrays of test cases with name, input, expected output

2. **Mock external services at the interface boundary**
   - Category: testing
   - Action: Create interface wrappers for external services, mock those interfaces
```

### Example 2: Before Writing Error Handling

Query: "error handling" or category=code-style tags=errors

```markdown
## Relevant Lessons for Error Handling

### High Confidence
1. **Wrap errors with context using fmt.Errorf**
   - Category: code-style
   - Action: Always add context when returning errors: `fmt.Errorf("failed to process user %d: %w", id, err)`

### Medium Confidence
2. **Use custom error types for domain errors**
   - Category: architecture
   - Note: Preferred for errors that callers need to handle specifically
```

## Integration Points

This skill can be used by:
- Code generation agents (query before writing)
- Code review agents (check compliance)
- Refactoring agents (apply patterns consistently)
- Documentation agents (follow comment conventions)

## No Lessons Found

If no lessons exist for this repository:

```markdown
## No Lessons Found

No review lessons have been captured for this repository yet.

Lessons are automatically extracted when using `/address-review` on PRs with feedback.
Consider the following general best practices for [category].
```
