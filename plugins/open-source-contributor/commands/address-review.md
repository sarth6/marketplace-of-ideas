---
description: Analyze the latest PR review comments and create a plan to address them
arguments:
  - name: pr_url
    description: Optional PR URL. If not provided, uses the PR for the current branch
    required: false
  - name: review_id
    description: Optional specific review ID to address. If not provided, uses the latest review
    required: false
---

# Address PR Review

You are an expert code reviewer assistant. Your task is to analyze the latest review on a PR and create a comprehensive plan to address all feedback.

## Input

- **PR URL**: $ARGUMENTS.pr_url (optional - defaults to current branch's PR)
- **Review ID**: $ARGUMENTS.review_id (optional - defaults to latest review)

---

## Phase 1: Gather PR Context 📋

First, establish full context about the PR and its changes.

### Step 1.1: Get PR Metadata

Run **in parallel** using Bash:

1. **Get PR details including reviews:**
   ```bash
   gh pr view $ARGUMENTS.pr_url --json number,title,body,url,headRefName,baseRefName,files,additions,deletions,reviews,latestReviews,comments,reviewDecision
   ```
   If `$ARGUMENTS.pr_url` is empty/not provided, run without the URL argument (uses current branch).

2. **Get the diff for the PR:**
   ```bash
   gh pr diff $ARGUMENTS.pr_url
   ```

### Step 1.2: Extract Key Information

From the PR metadata, extract:
- **PR Number**: For API calls
- **PR Title & Description**: Understanding the intent of the changes
- **Files Changed**: List of all modified files
- **Reviews Array**: All reviews with their states (APPROVED, CHANGES_REQUESTED, COMMENTED)
- **Latest Review**: The most recent review (or the specific one if `$ARGUMENTS.review_id` was provided)

---

## Phase 2: Fetch Review Comments 💬

### Step 2.1: Identify the Target Review

If `$ARGUMENTS.review_id` is provided, use that review ID. Otherwise:
1. From the PR metadata's `reviews` array, find the most recent review with state `CHANGES_REQUESTED` or `COMMENTED`
2. If no such review exists, use the absolute latest review
3. Extract the review ID

### Step 2.2: Get Review Comments

Using the GitHub API, fetch the review's line comments:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews/{review_id}/comments
```

**Note**: The `{owner}` and `{repo}` placeholders are automatically filled by `gh` based on the current repository context.

Also fetch any general (non-line-specific) review body from the review object itself.

### Step 2.3: Parse Comment Structure

For each review comment, extract:
- **path**: The file where the comment was left
- **line** / **original_line**: The line number (in new/old version)
- **diff_hunk**: The code context around the comment
- **body**: The actual feedback text
- **in_reply_to_id**: If this is a reply in a thread
- **side**: Whether it's on the LEFT (old) or RIGHT (new) side of the diff

Group comments by file path for organized exploration.

---

## Phase 3: Deep Codebase Exploration 🔍

For comprehensive understanding, deploy **parallel explore agents** using the Task tool with `subagent_type="Explore"`.

### Step 3.1: PR Context Exploration (run in parallel)

1. **PR Changes Agent**:
   - Prompt: "Explore the code changes in this PR. Files changed: [list files from Phase 1]. Understand what each change is trying to accomplish and how the changes relate to each other."

2. **Architecture Context Agent**:
   - Prompt: "Explore the architecture around these files: [list files with comments]. Understand the patterns, dependencies, and how these files fit into the larger codebase."

### Step 3.2: Comment-Specific Exploration (run in parallel, one per file with comments)

For each file that has review comments, create an explore agent:

- **File-specific Agent**:
  - Prompt: "Deeply explore [file_path]. Understand:
    1. The purpose of this file and its role in the codebase
    2. The specific code sections around lines [list line numbers from comments]
    3. Related code that might be affected by changes to these sections
    4. Testing patterns for this file/module

    Review comments on this file:
    [Include the actual comment bodies for context]"

### Step 3.3: Synthesize Understanding

After all explore agents complete, synthesize:
- What the reviewer is asking for in each comment
- The technical implications of each requested change
- Any potential ripple effects or related changes needed
- Conflicts or dependencies between different review comments

---

## Phase 4: Create Action Plan 📝

Now create a structured plan to address each review comment.

### Step 4.1: Categorize Comments

Organize comments into categories:

1. **Code Changes Required**: Comments requesting specific code modifications
2. **Questions to Answer**: Comments asking for clarification (may need PR description update or code comments)
3. **Design Discussions**: Comments about architectural decisions (may need discussion before action)
4. **Nits/Style**: Minor style or formatting suggestions
5. **Already Addressed**: Comments that may have been addressed in subsequent commits

### Step 4.2: Create Detailed Plan

For each actionable comment, create a plan entry:

```markdown
### Comment on [file_path]:[line_number]

**Reviewer said:** "[comment body]"

**Understanding:** [Your interpretation of what the reviewer wants]

**Proposed action:**
- [Specific changes to make]
- [Files that need to be modified]
- [Any related changes needed]

**Considerations:**
- [Potential impacts]
- [Questions to clarify with reviewer if any]
```

### Step 4.3: Determine Execution Order

Order the planned changes by:
1. Dependencies (changes that other changes depend on go first)
2. Risk level (safer changes first)
3. Logical grouping (related changes together)

---

## Phase 5: Enter Plan Mode 🎯

After completing your analysis, you MUST use the **EnterPlanMode** tool to transition into plan mode.

This is critical because:
1. It allows the user to review and approve your proposed changes before implementation
2. It provides a structured way to track progress through the changes
3. It ensures no changes are made without user consent

**Before calling EnterPlanMode**, output a summary:

```markdown
## Review Analysis Summary

**PR:** #[number] - [title]
**Review by:** [reviewer] ([state])
**Comments to address:** [count]

### Comment Overview
| File | Line | Category | Summary |
|------|------|----------|---------|
| ... | ... | ... | ... |

### Proposed Approach
[High-level description of how you'll address the review]

### Files to Modify
- [file1]: [brief description of changes]
- [file2]: [brief description of changes]

---

Entering plan mode to create detailed implementation plan...
```

Then call the **EnterPlanMode** tool to allow the user to review and approve the plan.

---

## Phase 6: Extract & Store Lessons 📚

After addressing review comments (when exiting plan mode or after implementation), extract generalizable insights that can help avoid similar feedback in the future.

### Step 6.1: Identify Lesson Candidates

Analyze each review comment for extractable lessons. Look for these **extraction signals**:

**Strong signals (high confidence):**
- Prescriptive language: "always", "never", "prefer", "should", "convention", "standard"
- Project standard references: "per our style guide", "we typically", "in this codebase"
- Pattern explanations: Comments that explain the "why" not just the "what"
- Repeated feedback: Same issue flagged across multiple files or comments

**Medium signals:**
- Clear maintainer preferences: "I'd prefer", "it's cleaner to"
- Best practice references: "idiomatically", "the Go way", "Pythonic"

**Skip these (not generalizable):**
- Typo corrections (one-off mistakes)
- Bug-specific fixes (unique to this code)
- Clarification questions without guidance
- Context-specific suggestions tied to unique business logic

### Step 6.2: Determine Storage Location

Use the repository's origin URL to create a consistent identifier across worktrees:

```bash
# Get repo identifier from origin URL
REPO_ID=$(git remote get-url origin 2>/dev/null | shasum -a 256 | cut -c1-12)

# Fallback to repo path if no remote configured
if [ -z "$REPO_ID" ]; then
  REPO_ID=$(git rev-parse --show-toplevel | shasum -a 256 | cut -c1-12)
fi

LESSONS_DIR="$HOME/.claude/projects/$REPO_ID"
mkdir -p "$LESSONS_DIR"
```

### Step 6.3: Check for Duplicates

Before adding a lesson, read the existing lessons file and check if a similar lesson already exists:

```bash
LESSONS_FILE="$LESSONS_DIR/review-lessons.md"
if [ -f "$LESSONS_FILE" ]; then
  # Read existing lessons to avoid duplicates
  cat "$LESSONS_FILE"
fi
```

If a similar lesson exists:
- If the new instance reinforces it, increment the **Occurrences** count
- If it adds nuance, update the lesson with the additional context
- If it's truly the same, skip adding

### Step 6.4: Categorize Lessons

Determine whether each lesson is **shared** (project conventions) or **personal** (individual preferences):

**Shared lessons** (`review-lessons.md`):
- Explicit project conventions or style guide rules
- Feedback from maintainers about project patterns
- Lessons with 3+ occurrences
- Security or performance requirements specific to the project

**Personal lessons** (`review-lessons.local.md`):
- Single-instance feedback that might be personal preference
- Style suggestions without clear project mandate
- Lessons you want to remember but aren't sure are universal

### Step 6.5: Format and Append Lessons

For each new lesson, format it as:

```markdown
### [Concise title describing the lesson]
- **Confidence:** [high|medium|low]
- **Source:** PR #[number], @[reviewer], [YYYY-MM-DD]
- **Category:** [code-style|architecture|testing|documentation|performance|security|conventions]
- **Tags:** [comma-separated relevant tags]
- **Occurrences:** 1

[Clear description of the lesson - what to do and why]

[Optional: code example showing before/after or correct pattern]

---
```

Append to the appropriate file:
```bash
# Append to shared or local lessons file
echo "$LESSON_CONTENT" >> "$LESSONS_DIR/review-lessons.md"
# OR
echo "$LESSON_CONTENT" >> "$LESSONS_DIR/review-lessons.local.md"
```

### Step 6.6: Update Repository Metadata

Create or update `repo-info.json` to track the repository:

```bash
cat > "$LESSONS_DIR/repo-info.json" << EOF
{
  "origin_url": "$(git remote get-url origin 2>/dev/null)",
  "repo_name": "$(basename $(git rev-parse --show-toplevel))",
  "last_accessed": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "lessons_count": $(grep -c "^### " "$LESSONS_DIR/review-lessons.md" 2>/dev/null || echo 0)
}
EOF
```

### Step 6.7: Report to User

After extracting lessons, report a summary:

```markdown
## 📚 Lessons Learned

**Extracted [N] new lessons from this review:**

| Lesson | Category | Confidence |
|--------|----------|------------|
| [title] | [category] | [confidence] |

**Storage location:** `~/.claude/projects/[repo-hash]/review-lessons.md`

💡 Use `/review-lessons list` to view all lessons for this project.
💡 Use `/review-lessons search <query>` to find specific lessons.
```

---

## Error Handling

- **No PR found for current branch**: If `gh pr view` fails because there's no PR, inform the user and ask them to provide a PR URL or create a PR first.

- **No reviews found**: If the PR has no reviews yet, inform the user:
  ```
  ℹ️ This PR doesn't have any reviews yet.
  Would you like me to:
  1. Help you request a review from someone?
  2. Do a self-review of the changes?
  ```

- **Review has no comments**: If the review exists but has no line comments (only an approval or general comment), report this and show the general review body if present.

- **API rate limiting**: If GitHub API calls fail due to rate limiting, inform the user and suggest waiting or using a different authentication method.

- **Permission errors**: If the user doesn't have access to the repository or PR, clearly explain the access issue.

---

## Output Format

Throughout execution, provide clear progress updates:

```
📋 Phase 1: Gathering PR context...
   ✓ PR #123: "Add new feature X"
   ✓ 5 files changed (+150, -30)

💬 Phase 2: Fetching review comments...
   ✓ Found review by @reviewer (CHANGES_REQUESTED)
   ✓ 7 comments across 3 files

🔍 Phase 3: Exploring codebase...
   ✓ Launched 5 parallel explore agents
   ✓ [Agent results summary]

📝 Phase 4: Creating action plan...
   ✓ Categorized 7 comments
   ✓ Created implementation plan

🎯 Phase 5: Entering plan mode...

📚 Phase 6: Extracting lessons learned...
   ✓ Found [N] lesson candidates
   ✓ Saved to ~/.claude/projects/[repo-hash]/review-lessons.md
```
