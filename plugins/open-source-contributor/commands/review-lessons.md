---
description: Manage lessons learned from PR reviews - list, search, consolidate, and promote lessons
arguments:
  - name: action
    description: "Action to perform: list, search, consolidate, or promote"
    required: true
  - name: query
    description: "For search: text or category=value or tags=value to filter by"
    required: false
  - name: lesson_id
    description: "For promote: the lesson title or index to promote from personal to shared"
    required: false
---

# Review Lessons Management

Manage the lessons learned from PR review feedback. Lessons are stored in `~/.claude/projects/<repo-hash>/` and shared across all worktrees of the same repository.

## Input

- **Action**: $ARGUMENTS.action (required) - One of: `list`, `search`, `consolidate`, `promote`
- **Query**: $ARGUMENTS.query (optional) - Search query or filter
- **Lesson ID**: $ARGUMENTS.lesson_id (optional) - Lesson to promote

---

## Step 1: Determine Storage Location

First, locate the lessons directory for this repository:

```bash
# Get repo identifier from origin URL (same as address-review uses)
REPO_ID=$(git remote get-url origin 2>/dev/null | shasum -a 256 | cut -c1-12)

# Fallback to repo path if no remote configured
if [ -z "$REPO_ID" ]; then
  REPO_ID=$(git rev-parse --show-toplevel | shasum -a 256 | cut -c1-12)
fi

LESSONS_DIR="$HOME/.claude/projects/$REPO_ID"
SHARED_FILE="$LESSONS_DIR/review-lessons.md"
LOCAL_FILE="$LESSONS_DIR/review-lessons.local.md"
```

If the directory doesn't exist, report that no lessons have been captured yet.

---

## Action: `list`

Display all lessons organized by category.

### Step 2a: Read and Parse Lessons

Read both the shared and local lessons files:

```bash
echo "=== SHARED LESSONS ==="
cat "$SHARED_FILE" 2>/dev/null || echo "(no shared lessons yet)"

echo ""
echo "=== PERSONAL LESSONS ==="
cat "$LOCAL_FILE" 2>/dev/null || echo "(no personal lessons yet)"
```

### Step 3a: Format Output

Present lessons in a readable format grouped by category:

```markdown
## 📚 Review Lessons for [repo-name]

**Storage:** `~/.claude/projects/[repo-hash]/`

### Shared Lessons (review-lessons.md)

#### Code Style
| # | Lesson | Confidence | Occurrences |
|---|--------|------------|-------------|
| 1 | Use early returns for guard clauses | high | 3 |
| 2 | Prefer const over let when possible | medium | 1 |

#### Architecture
| # | Lesson | Confidence | Occurrences |
|---|--------|------------|-------------|
| 3 | Keep handlers thin, move logic to services | high | 2 |

### Personal Lessons (review-lessons.local.md)

#### Code Style
| # | Lesson | Confidence | Occurrences |
|---|--------|------------|-------------|
| 4 | Reviewer prefers explicit type annotations | low | 1 |

---

**Total:** [N] shared lessons, [M] personal lessons
```

---

## Action: `search`

Find lessons matching a query.

### Step 2b: Parse Search Query

The query can be:
- **Free text**: Search in lesson titles and descriptions
- **category=VALUE**: Filter by category (code-style, architecture, testing, etc.)
- **tags=VALUE**: Filter by tags (comma-separated)
- **confidence=VALUE**: Filter by confidence level (high, medium, low)

### Step 3b: Search Lessons

```bash
# Combine both files for searching
cat "$SHARED_FILE" "$LOCAL_FILE" 2>/dev/null | grep -i "$QUERY"
```

For structured queries, parse the lessons and filter accordingly.

### Step 4b: Format Results

```markdown
## 🔍 Search Results for "[query]"

Found [N] matching lessons:

### 1. Use early returns for guard clauses
- **Confidence:** high | **Category:** code-style
- **Source:** PR #142, @reviewer, 2025-01-15
- **File:** review-lessons.md (shared)

Prefer early returns for validation rather than nested if-else.

---

### 2. [Next matching lesson...]
```

If no results found:
```markdown
## 🔍 Search Results for "[query]"

No lessons found matching your query.

**Suggestions:**
- Try a broader search term
- Check available categories: code-style, architecture, testing, documentation, performance, security, conventions
- Use `/review-lessons list` to see all lessons
```

---

## Action: `consolidate`

Merge similar lessons to reduce duplication and increase confidence.

### Step 2c: Identify Similar Lessons

Read all lessons and identify candidates for consolidation:
- Lessons with similar titles (fuzzy match)
- Lessons in the same category with overlapping tags
- Lessons that describe the same underlying principle

### Step 3c: Present Consolidation Candidates

```markdown
## 🔄 Consolidation Candidates

### Group 1: Guard Clause Patterns
These lessons appear related and could be merged:

| # | Lesson | Source | Confidence |
|---|--------|--------|------------|
| 1 | Use early returns for guard clauses | PR #142 | high |
| 5 | Prefer early return over nested conditionals | PR #167 | medium |

**Suggested merged lesson:**
> Use early returns for guard clauses instead of nested conditionals

**New confidence:** high (multiple sources)
**Combined occurrences:** 4

---

### Group 2: [Next group...]
```

### Step 4c: Apply Consolidation (with user confirmation)

Use the **AskUserQuestion** tool to confirm each consolidation:

> Should I merge these lessons into a single entry? The original lessons will be preserved in a `# Consolidated` comment.

If confirmed:
1. Create the merged lesson with combined metadata
2. Comment out or remove the original lessons
3. Update occurrence counts

---

## Action: `promote`

Move a lesson from personal (review-lessons.local.md) to shared (review-lessons.md).

### Step 2d: Find the Lesson

If `$ARGUMENTS.lesson_id` is provided, find that specific lesson. Otherwise, list personal lessons and ask which to promote.

### Step 3d: Confirm Promotion

Use **AskUserQuestion** tool:

```markdown
**Promoting lesson to shared:**

### [Lesson title]
- **Current confidence:** [level]
- **Category:** [category]

[Lesson content]

This will move the lesson from `review-lessons.local.md` to `review-lessons.md`, making it visible as a project convention.

Confirm promotion?
```

### Step 4d: Execute Promotion

If confirmed:

1. Read the lesson from the local file
2. Optionally increase confidence level (low → medium, medium → high)
3. Append to the shared file
4. Remove from the local file
5. Update repo-info.json

```bash
# After promotion
echo "✅ Lesson promoted to shared lessons"
echo "   From: review-lessons.local.md"
echo "   To: review-lessons.md"
```

---

## Error Handling

- **No lessons directory**: Report that no lessons have been captured yet and suggest running `/address-review` on a PR with feedback.

- **Empty lessons files**: Report that the files exist but are empty.

- **Invalid action**: Show available actions:
  ```
  ❌ Unknown action: [provided action]

  Available actions:
  - list: Display all lessons by category
  - search <query>: Find lessons matching query
  - consolidate: Merge similar lessons
  - promote <lesson>: Move personal → shared
  ```

- **Lesson not found (for promote)**: Show available personal lessons and ask user to specify which one.

---

## Output Format

Always include the storage location for transparency:

```
📚 Lessons stored at: ~/.claude/projects/[repo-hash]/
   Repo: [origin URL or path]
   Last updated: [date]
```
