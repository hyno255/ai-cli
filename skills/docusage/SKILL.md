---
name: docusage
description: Analyze how one service is used by another. Documents the why, how, and where of cross-service interactions including triggers, call chains, and data flow. Use when you need to understand how serviceB depends on serviceA.
disable-model-invocation: true
argument-hint: <serviceA-path> <serviceB-path> [additional context]
---

# Cross-Service Usage Analysis Orchestrator

You are orchestrating a multi-phase analysis of how one service uses another. The goal is to document the "why, how, and where" of cross-service interactions so engineers can understand the end-to-end dependency.

**Arguments:**
- `$0` = **required** path to Service A (the service being consumed)
- `$1` = **required** path to Service B (the consumer)
- Everything after = **optional** additional context (e.g., "serviceA exposes a gRPC API", "check the event bus")

**Service A (consumed):** `$0`
**Service B (consumer):** `$1`
**Additional context:** `$ARGUMENTS`

## Before You Start

1. Verify both service paths exist
2. If `.ai-doc/` doesn't exist in Service A, suggest adding `.ai-doc/` to `.gitignore`
3. Create workspace: `$0/.ai-doc/docusage/`

---

## Phase 1 -- Interface Analysis

**Goal:** Understand Service A's public interface -- what it exposes that others can consume.

Spawn a `researcher` subagent:

> Analyze the public interface of the service at `$0`.
> Read the tracing guide at `skills/docusage/tracing-guide.md` for what to look for.
> Additional context from the user: $ARGUMENTS
>
> Write your analysis to: `$0/.ai-doc/docusage/interfaces.md`
>
> Identify and document:
> 1. **Exported APIs**: REST endpoints, gRPC services, GraphQL queries/mutations
> 2. **Events emitted**: Messages published to queues/topics, webhooks fired
> 3. **Shared data models**: Database tables accessed by multiple services, shared schemas
> 4. **Configuration hooks**: Environment variables, feature flags, config files that affect behavior
> 5. **SDK/client libraries**: If Service A provides a client library, document its interface
>
> For each interface point, note: what it does, what data it accepts/returns, and any important behavior (rate limits, auth requirements, error codes).

**After the researcher returns:**
- Check for `## OVERFLOW`. If found, spawn additional researchers for uncovered interface areas.

---

## Phase 2 -- Usage Tracing

**Goal:** Find every place Service B interacts with Service A and understand why.

Spawn a `researcher` subagent:

> Scan the service at `$1` (Service B) for all interactions with the service at `$0` (Service A).
> Read the tracing guide at `skills/docusage/tracing-guide.md` for patterns to search for.
> Also read `$0/.ai-doc/docusage/interfaces.md` to know what Service A exposes.
> Additional context from the user: $ARGUMENTS
>
> Write your analysis to: `$0/.ai-doc/docusage/used-by-$1/analysis.md`
> (Replace $1 with the service directory name, e.g., `used-by-order-service/analysis.md`)
>
> For each interaction found, document:
> 1. **What is called**: Which API/event/data model of Service A is used
> 2. **Where in Service B**: File path and function that makes the call
> 3. **Why (business reason)**: What business need drives this interaction
> 4. **Trigger**: What event in Service B triggers the call to Service A
> 5. **Data flow**: What data goes from B to A, and what comes back
> 6. **Error handling**: What happens in B if the call to A fails
>
> Group findings by workflow/feature. Name each group clearly (e.g., "Order Validation", "User Lookup During Checkout").

**After the researcher returns:**
- Check for `## OVERFLOW`. If found, spawn additional researchers for uncovered areas of Service B.

---

## Phase 3 -- Synthesis

**Goal:** Generate the final usage documentation.

Read all files in `$0/.ai-doc/docusage/`.

Spawn a `doc-writer` subagent:

> Generate usage documentation for how `$1` uses `$0`.
>
> **Input files:**
> - Service A interface: `$0/.ai-doc/docusage/interfaces.md`
> - Usage analysis: `$0/.ai-doc/docusage/used-by-<serviceB>/analysis.md`
>
> **Templates** (use as structural guides):
> - Usage overview: `skills/docusage/templates/usage-overview.md`
> - Usage workflow: `skills/docusage/templates/usage-workflow-doc.md`
>
> **Output directory:** `$0/docs/<serviceA-name>/usage/`
>
> **Output structure:**
> ```
> docs/<serviceA>/usage/
> ├── README.md              # Index of all consumers with summary
> └── <serviceB>/
>     ├── README.md          # Overview: why B uses A, dependency diagram
>     └── workflows/
>         ├── README.md      # Workflow index
>         └── <workflow>.md  # Per-workflow: trigger, call chain, data flow
> ```
>
> Key rules:
> - Lead with "why B uses A" -- the business reason, not just the technical mechanism
> - Include a mermaid dependency diagram in the consumer overview
> - Each workflow doc should have a sequence diagram showing the interaction
> - Cross-reference back to Service A's interface docs if they exist
> - Keep files under ~300 lines, split if needed

**After the writer returns:**
- Check for `## OVERFLOW`. If found, spawn additional writers.
- Verify output structure is complete with README.md indexes.

---

## Phase 4 -- Auto-Review (Agent Team)

**Goal:** Review and fix the usage docs in one collaborative round.

Create an agent team with 3 teammates:

> Create an agent team to review and revise the usage documentation in `$0/docs/<serviceA>/usage/`.
>
> **Reviewer teammate:** Read all docs in the output directory. Check against `skills/doc-review/review-checklist.md`. Also check `$0/docs/review-notes.md` if it exists. Triage issues as "needs research" or "revision only". Message teammates directly.
>
> **Researcher teammate:** When the reviewer identifies issues needing codebase investigation, research code at `$0` and/or `$1`. Share findings with the writer.
>
> **Writer teammate:** Revise affected doc files based on reviewer and researcher feedback.
>
> Complete after one round. Ephemeral -- do not persist review feedback.

Wait for the team to finish, then clean up.

---

## Done

Summarize what was generated:
- List the doc files created
- Summarize the key interactions found between the two services
- Note any areas where coverage may be thin
- Remind the user they can run `/ai-cli:doc-review` to provide feedback
