---
name: doc-gen
description: Generate comprehensive documentation for a codebase. Analyzes project structure, workflows, and APIs, then produces well-structured docs. Use when you need to document a repo or service.
disable-model-invocation: true
argument-hint: <path> [additional context]
---

# Documentation Generation Orchestrator

You are orchestrating a multi-phase documentation generation process. Follow each phase in order. Use the file system as working memory between phases.

**Arguments:**
- `$ARGUMENTS[0]` = **required** path to the repo/service to document
- Everything after the path = **optional** additional context (API doc locations, focus areas, tech stack hints)

**Target path:** `$0`
**Additional context from user:** `$ARGUMENTS`

## Before You Start

1. Verify the target path exists and contains code
2. If `.ai-doc/` doesn't exist yet, suggest adding `.ai-doc/` to `.gitignore`
3. Create the workspace directory: `<target>/.ai-doc/do c/`

---

## Phase 1 -- Discovery

**Goal:** Produce a high-level inventory of the codebase.

Spawn a `researcher` subagent with this task:

> Scan the codebase at `$0` and produce a structured inventory.
> Read the analysis guide at `skills/doc-gen/analysis-guide.md` for what to look for.
> Additional context from the user: $ARGUMENTS
>
> Write two files:
> 1. `$0/.ai-doc/doc-gen/discovery/repo-map.md` -- A structured overview of the repo: directory structure, tech stack, key entry points, external dependencies, architectural pattern (microservices, monolith, etc.)
> 2. `$0/.ai-doc/doc-gen/discovery/components.md` -- A list of identified components/modules, one per section, with: name, path, 1-2 sentence description, and estimated complexity (small/medium/large)
>
> Focus on identifying the major building blocks and how they relate. Do not deep-dive into any single component yet.

**After the researcher returns:**
- Read `$0/.ai-doc/doc-gen/discovery/components.md`
- Check for a `## OVERFLOW` section. If present, spawn additional researchers scoped to the uncovered areas
- **Decision gate:** If fewer than ~5 components are identified, combine Phase 1 and Phase 2 by asking the researcher to also do deep analysis

---

## Phase 2 -- Deep Analysis

**Goal:** Produce detailed analysis for each identified component.

Read `$0/.ai-doc/doc-gen/discovery/components.md` to get the list of components.

For each component, spawn a `researcher` subagent (up to 4 in parallel):

> Perform a deep analysis of the component at `<component-path>` within `$0`.
> Read the analysis guide at `skills/doc-gen/analysis-guide.md` for the expected output format.
> Additional context from the user: $ARGUMENTS
>
> Write your analysis to: `$0/.ai-doc/doc-gen/analysis/<component-name>.md`
>
> Cover: purpose, responsibilities, key interfaces, dependencies (in/out), participation in workflows, notable design decisions. Focus on "why" over "what". Reference code by file path and line range -- do not paste large blocks.

**After each researcher returns:**
- Check for `## OVERFLOW` sections in the output files
- If overflow is found, spawn additional researchers scoped to the uncovered areas
- Repeat until all components are fully analyzed

---

## Phase 3 -- Synthesis

**Goal:** Generate the final documentation from all analysis files.

Read all files in `$0/.ai-doc/doc-gen/analysis/` and `$0/.ai-doc/doc-gen/discovery/repo-map.md`.

Spawn a `doc-writer` subagent:

> Generate documentation for the project analyzed in `$0`.
>
> **Input files:**
> - Project overview: `$0/.ai-doc/doc-gen/discovery/repo-map.md`
> - Component analyses: all `.md` files in `$0/.ai-doc/doc-gen/analysis/`
>
> **Templates** (use as structural guides):
> - Project README: `skills/doc-gen/templates/project-overview.md`
> - Workflow doc: `skills/doc-gen/templates/workflow-doc.md`
> - API doc: `skills/doc-gen/templates/api-doc.md`
>
> **Output directory:** `$0/docs/`
>
> **Output structure:**
> ```
> docs/
> ├── README.md              # Project overview with architecture diagram
> ├── workflows/
> │   ├── README.md          # Workflow index
> │   └── <workflow>.md      # Per-workflow docs (components explained in context)
> └── api/
>     ├── README.md          # API index grouped by domain
>     └── <api-group>.md     # Per-API-group with behavior + workflow cross-refs
> ```
>
> Key rules:
> - No separate architecture/ folder. Components are explained within the workflows they participate in.
> - The project README.md covers architecture at a high level with a mermaid diagram.
> - API docs describe behavior and cross-reference the workflows where each endpoint is triggered.
> - Keep each file under ~300 lines. Split into folders with README.md index if larger.
> - Every folder must have a README.md as index.

**After the writer returns:**
- Check for `## OVERFLOW` sections. If found, spawn additional writers for the remaining sections.
- Verify that all expected output files exist and every folder has a README.md.

---

## Phase 4 -- Auto-Review (Agent Team)

**Goal:** Review the generated docs and fix issues in one collaborative round.

Create an agent team with 3 teammates:

> Create an agent team to review and revise the documentation just generated in `$0/docs/`.
>
> **Reviewer teammate:** Read all docs in `$0/docs/`. Check against the quality checklist at `skills/doc-review/review-checklist.md`. Also check `$0/docs/review-notes.md` if it exists for persistent human feedback. For each issue found, triage it as "needs research" (send to researcher) or "revision only" (send to writer). Message teammates directly with specific instructions.
>
> **Researcher teammate:** When the reviewer identifies issues that need codebase investigation, research the code at `$0` and share your findings with the writer teammate. Update analysis files in `$0/.ai-doc/` if the underlying analysis was wrong or incomplete.
>
> **Writer teammate:** When the reviewer or researcher sends you revision instructions, update the affected doc files in `$0/docs/`. Follow the same writing conventions: lead with why, reference code by path, keep files under 300 lines.
>
> Complete after one round of review and fixes. Do not persist review feedback -- this is an ephemeral quality pass.

Wait for the team to finish, then clean up.

---

## Done

Summarize what was generated:
- List the doc files created in `$0/docs/`
- Note any areas where coverage may be thin (from overflow handling)
- Remind the user they can run `/ai-cli:doc-review` to provide feedback for targeted revisions
