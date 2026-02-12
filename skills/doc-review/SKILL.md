---
name: doc-review
description: Submit feedback on generated documentation and trigger targeted revisions. Persists human feedback for future auto-reviews. Use when you find issues in generated docs or want to improve them.
disable-model-invocation: true
argument-hint: <feedback about what to fix or improve>
---

# Documentation Review & Revision Orchestrator

You are orchestrating a human-feedback-driven revision of generated documentation. The user has provided specific feedback about issues they found in the docs.

**Arguments:**
- `$ARGUMENTS` = the user's feedback (what to fix, what's wrong, what's missing)

**User feedback:** $ARGUMENTS

## Step 1 -- Persist Human Feedback

First, determine which service/project the feedback applies to by examining the feedback text and the current working directory. Look for an existing `docs/` directory.

Append the user's feedback to `docs/review-notes.md` (create the file if it doesn't exist):

```markdown
---
date: [current date/time]
---
$ARGUMENTS
```

This file is persistent -- it accumulates over time. Future `/ai-cli:docgen` and `/ai-cli:docusage` runs will load this file during their Phase 4 auto-review, so the same issues won't recur.

Tell the user: "Feedback persisted to `docs/review-notes.md`. This will be checked in all future auto-reviews."

## Step 2 -- Create Revision Agent Team

Create an agent team with 3 teammates to address the feedback:

> Create an agent team to revise documentation based on human feedback.
>
> **The user's feedback is:**
> $ARGUMENTS
>
> **Reviewer teammate:** Read the user's feedback above. Examine the current docs in `docs/`. For each issue in the feedback, determine:
> - Is it a "needs research" issue? (missing information, wrong facts, gaps that require looking at the code)
> - Is it a "revision only" issue? (style, structure, presentation, wording)
> Then message the appropriate teammate:
> - For research issues: message the researcher with exactly what to investigate and where to look
> - For revision issues: message the writer with exactly what to change and in which file
>
> **Researcher teammate:** When the reviewer sends you an investigation request:
> 1. Explore the codebase to find the missing or incorrect information
> 2. Update or create analysis files in `.ai-doc/` with your findings
> 3. Message the writer with your findings and which doc files need updating
>
> **Writer teammate:** When you receive instructions from the reviewer (for simple fixes) or findings from the researcher (for content fixes):
> 1. Read the affected doc files
> 2. Make the targeted revisions -- only change what's needed, preserve everything else
> 3. Follow writing conventions: lead with why, reference code by path, keep files under 300 lines
>
> Complete after one round of review + fix. The user's feedback has already been persisted to `docs/review-notes.md` -- do not modify that file.

Wait for the team to finish, then clean up.

## Step 3 -- Report

After the team finishes, summarize:
- What issues were addressed
- Which files were modified
- Whether any issues could not be resolved (and why)
- Remind the user they can run `/ai-cli:doc-review` again with additional feedback
