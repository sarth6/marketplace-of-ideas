---
description: Analyze a GitHub issue and produce a comprehensive design doc for implementing it
arguments:
  - name: issue_url
    description: URL to the GitHub issue (e.g., https://github.com/owner/repo/issues/123)
    required: true
  - name: interactive
    description: Enable interactive mode with checkpoints for user input
    required: false
    type: boolean
    default: false
---

# Open Source Issue Solver

You are an expert open source contributor. Your task is to analyze a GitHub issue and produce a comprehensive design document that would enable someone to implement the solution.

## Input

- **Issue URL**: $ARGUMENTS.issue_url
- **Interactive Mode**: $ARGUMENTS.interactive

## Execution Strategy

<autonomous_mode>
When `interactive` is false (default), execute all phases without interruption, making reasonable decisions based on your analysis. Only ask questions if you encounter a genuine blocker.
</autonomous_mode>

<interactive_mode>
When `interactive` is true, pause after each major phase for user feedback:
1. After Phase 1: Confirm understanding of the issue
2. After Phase 2: Validate exploration findings
3. After Phase 3: Review design approach before writing
4. After Phase 5: Get feedback on the draft before finalizing
</interactive_mode>

---

## Phase 1: Issue Analysis 📋

First, fetch and deeply understand the GitHub issue:

1. **Use WebFetch** to read the issue at `$ARGUMENTS.issue_url`
2. **Extract key information:**
   - Issue title and number
   - Core problem being solved
   - Any constraints or requirements mentioned
   - Acceptance criteria (explicit or implied)
   - Referenced PRs, issues, or discussions
   - Key participants and their concerns

3. **Identify complexity signals:**
   - Are there long comment threads? (indicates nuance/controversy)
   - Are there linked PRs that failed? (learn from their mistakes)
   - Are there competing approaches suggested?
   - What's the maintainer's stance?

If the issue references other PRs or issues, fetch those too to understand the full context.

---

## Phase 2: Codebase Exploration 🔍

Deploy parallel explore agents to understand the codebase context:

1. **Architecture Agent**: Understand the overall project structure
   - What are the main modules/packages?
   - What patterns does the codebase follow?
   - Where would this feature likely live?

2. **Related Code Agent**: Find code related to this issue
   - Search for keywords from the issue
   - Find similar existing features
   - Identify integration points

3. **Testing Patterns Agent**: Understand how to test the feature
   - What testing frameworks are used?
   - What's the test coverage expectation?
   - Are there similar tests to reference?

4. **API Surface Agent**: If this affects public API
   - What's the current API style?
   - How are breaking changes handled?
   - What's the deprecation policy?

Use the Task tool with subagent_type="Explore" for each agent. Run them in parallel for efficiency.

---

## Phase 3: Research & Ideation 💡

If relevant, research how other projects solve similar problems:

1. **Use WebSearch** to find:
   - How similar libraries handle this feature
   - Best practices and common patterns
   - Potential pitfalls to avoid

2. **Analyze tradeoffs:**
   - Performance vs. simplicity
   - Flexibility vs. ease of use
   - Breaking changes vs. backwards compatibility

3. **If there are failed PRs**, analyze what went wrong:
   - What feedback did reviewers give?
   - What approaches were tried?
   - What should we do differently?

---

## Phase 4: Design Document Planning 📝

Structure your design document with these sections:

```markdown
# Design Document: [Feature Name]

## Overview
Brief summary of what we're building and why.

## Problem Statement
- What problem does this solve?
- Who benefits from this?
- What's the current workaround (if any)?

## Goals & Non-Goals
### Goals
- Specific, measurable objectives

### Non-Goals
- What this design explicitly does NOT address

## Proposed Solution

### High-Level Design
Architecture diagrams, data flow, key components.

### Detailed Design
- API surface (with code examples)
- Internal implementation approach
- Key data structures
- Integration points

### Code Examples
Show how users will use this feature:
```python
# Before (current workaround)
...

# After (with new feature)
...
```

## Alternatives Considered
Document other approaches and why they weren't chosen.

## Implementation Plan
Phased approach with clear milestones:
1. Phase 1: [Scope] - What gets built first
2. Phase 2: [Scope] - Next iteration
...

## Testing Strategy
- Unit tests needed
- Integration tests needed
- Manual testing scenarios

## Migration & Backwards Compatibility
- Breaking changes (if any)
- Migration path for existing users
- Deprecation timeline

## Open Questions
Things that need resolution before/during implementation.

## References
- Links to issues, PRs, discussions
- External documentation
```

---

## Phase 5: Write the Design Document ✍️

Using all your gathered context, write a comprehensive design document that:

1. **Is actionable**: Someone could implement from this doc
2. **Is complete**: Covers edge cases and error handling
3. **Is honest**: Acknowledges tradeoffs and uncertainties
4. **Is clear**: Assumes reader knows the project basics but explains your reasoning

**Important writing guidelines:**
- Use code examples liberally - they clarify intent better than prose
- Reference specific files/functions in the codebase where relevant
- Include file paths where new code should live
- Explain WHY you made certain design choices, not just WHAT

---

## Phase 6: Self-Review 🔎

Before finalizing, review your document:

1. **Completeness check:**
   - Would a contributor have enough info to start coding?
   - Are edge cases addressed?
   - Is error handling considered?

2. **Clarity check:**
   - Are there confusing parts that need more explanation?
   - Would code examples help clarify anything?
   - Is the scope clear (what's in vs. out)?

3. **Feasibility check:**
   - Does this fit the codebase's patterns?
   - Is the implementation plan realistic?
   - Are there dependencies or blockers?

Make improvements based on your review.

---

## Phase 7: Output 📄

Extract the issue number from the URL and save the design document:

1. Parse the issue number from `$ARGUMENTS.issue_url` (e.g., `123` from `.../issues/123`)
2. Use the **Write tool** to save the document as `design-doc-issue-{number}.md` in the current working directory
3. Provide a brief summary of what was created

**Final output format:**
```
✅ Design document created: design-doc-issue-{number}.md

Summary:
- [Brief description of the proposed solution]
- [Key design decisions made]
- [Recommended next steps]
```

---

## Error Handling

- If the issue URL is invalid or inaccessible, inform the user and ask for a correct URL
- If the codebase exploration reveals the feature already exists, report this finding
- If there are conflicting requirements in the issue discussion, highlight these in the design doc's "Open Questions" section
