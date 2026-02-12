# Documentation Quality Checklist

Use this checklist when reviewing generated documentation. Focus on issues that would confuse or mislead readers. Do not flag minor style preferences.

## High Severity (must fix)

- [ ] **Every key workflow is documented.** If the codebase has a major user-facing flow (auth, checkout, data pipeline, etc.), there should be a workflow doc for it.
- [ ] **No incorrect information.** Facts in the docs should match the actual code. Watch for outdated API descriptions, wrong file paths, or incorrect architectural claims.
- [ ] **"Why" before "what."** Every workflow and component section should explain the business reason and purpose before describing the implementation.
- [ ] **No code dumps.** No more than 5 lines of code should be pasted into any doc. Longer code should be referenced by file path and line range.
- [ ] **Consistent with codebase.** API endpoints, function names, and component names in the docs should match what's actually in the code.

## Medium Severity (should fix)

- [ ] **All cross-references are valid.** Every markdown link should point to a file that actually exists. Check relative paths especially.
- [ ] **No unexplained jargon.** Technical terms and project-specific names should be explained on first use. Acronyms should be expanded.
- [ ] **Complete indexes.** Every folder with multiple docs should have a `README.md` that links to all files in that folder.
- [ ] **Workflow docs have sequence diagrams.** Multi-step workflows should include a mermaid sequence diagram showing the key interactions.
- [ ] **API docs cross-reference workflows.** Each API endpoint should link to the workflow(s) where it's triggered.
- [ ] **No orphan docs.** Every doc file should be reachable from the root `docs/README.md` through index links.

## Low Severity (nice to have)

- [ ] **Files under 300 lines.** Long files should be split into folders with README.md indexes.
- [ ] **Architecture diagram in project README.** The main `docs/README.md` should have a mermaid diagram showing the high-level architecture.
- [ ] **Getting started pointers.** The project README should tell a newcomer where to look first.
- [ ] **Error handling documented.** Workflows should mention what happens when things fail.

## Checking Persistent Feedback

If `docs/review-notes.md` exists, also check:

- [ ] **Past issues don't recur.** Read each entry in `review-notes.md` and verify the current docs don't have the same problem.
- [ ] **Feedback was addressed.** If review notes mention specific missing content, check that it's now present.

## Review Output Format

For each issue found, report:

1. **Severity**: HIGH / MEDIUM / LOW
2. **File**: exact path to the affected doc file
3. **Section**: which heading or area has the issue
4. **Issue**: what's wrong (be specific)
5. **Fix type**: "research" (need to re-examine code) or "revision" (just rewrite the doc)
6. **Suggested fix**: concrete suggestion for how to fix it
