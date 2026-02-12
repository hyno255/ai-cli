# ai-cli

A Claude Code plugin for AI-native documentation generation and cross-service usage analysis. 

## What It Does

- **`/ai-cli:doc-gen`** -- Analyze a codebase and generate docs covering architecture, workflows, and APIs
- **`/ai-cli:doc-usage`** -- Analyze how one service is used by another: why, how, where, and what triggers the interactions
- **`/ai-cli:doc-review`** -- Submit feedback on generated docs, triggering targeted research and revision

## Prerequisites

- [Claude Code](https://code.claude.com) v1.0.33+
- Agent teams enabled (for the auto-review phase):
  ```json
  // Add to your Claude Code settings.json
  {
    "env": {
      "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
    }
  }
  ```

## Installation

Load the plugin by pointing Claude Code at this directory:

```bash
claude --plugin-dir /path/to/ai-cli
```

After loading, the three skills appear as slash commands: `/ai-cli:doc-gen`, `/ai-cli:doc-usage`, and `/ai-cli:doc-review`.

## Usage

### Generate Documentation

```
/ai-cli:doc-gen ./path/to/service
```

With optional context:

```
/ai-cli:doc-gen ./services/auth The OpenAPI spec is at api/openapi.yaml, focus on OAuth flows
/ai-cli:doc-gen . This is a Python Django project, main app is in src/
```

**What it does:**
1. **Discovery** -- Scans the repo structure, identifies components, APIs, and workflows
2. **Deep Analysis** -- Analyzes each component in detail (parallelized for large repos)
3. **Synthesis** -- Generates docs organized by workflow, with API references
4. **Auto-Review** -- A review team checks for quality issues and fixes them

**Output:** `docs/` directory with project overview, workflow docs, and API reference.

### Analyze Cross-Service Usage

```
/ai-cli:doc-usage ./services/auth ./services/orders
```

With optional context:

```
/ai-cli:doc-usage ./services/auth ./services/orders Check the event bus integration
```

**What it does:**
1. **Interface Analysis** -- Maps Service A's public API, events, and data models
2. **Usage Tracing** -- Finds every interaction from Service B to Service A, grouped by workflow
3. **Synthesis** -- Generates usage docs showing why, how, and where B depends on A
4. **Auto-Review** -- Quality check and fix

**Output:** `docs/<serviceA>/usage/<serviceB>/` with overview and per-workflow interaction docs.

### Review and Revise Docs

```
/ai-cli:doc-review The workflow docs for auth are missing the OAuth refresh token flow
/ai-cli:doc-review The API docs should mention rate limiting behavior for all endpoints
```

**What it does:**
1. Persists your feedback to `docs/review-notes.md` (loaded by all future auto-reviews)
2. Creates an agent team (reviewer + researcher + writer) to address your feedback
3. The team triages issues, researches code if needed, and revises the affected docs

## How It Works

This is a pure agent + skill plugin. There is no application code -- everything is markdown files that instruct Claude Code's agents.

### Architecture

```
ai-cli/
├── .claude-plugin/plugin.json     # Plugin manifest
├── skills/
│   ├── doc-gen/                    # /ai-cli:doc-gen orchestrator + templates
│   ├── doc-usage/                  # /ai-cli:doc-usage orchestrator + templates
│   └── doc-review/                # /ai-cli:doc-review orchestrator + checklist
└── agents/
    ├── researcher.md              # Read-only codebase analysis
    ├── doc-writer.md              # Documentation writer (conventions embedded)
    └── doc-reviewer.md            # Quality reviewer
```

### Key Design Decisions

- **Workflows over components**: Docs are organized by workflow. Components are explained within the workflows they participate in -- this gives readers the "why" naturally.
- **File system as memory**: Intermediate analysis files go to `.ai-doc/` inside each service. Phases read/write these files, keeping context windows manageable.
- **Overflow protocol**: If a repo is too large for one agent pass, agents self-report what they couldn't cover and the orchestrator spawns additional focused agents. No information is lost.
- **Agent teams for review**: The auto-review phase creates a team of reviewer + researcher + writer that communicate directly, enabling parallel triage and fix.
- **Persistent human feedback**: Review notes in `docs/review-notes.md` accumulate over time and are checked by all future auto-reviews.

### Working Memory

Analysis files are stored inside the service being analyzed:

```
<service>/.ai-doc/
├── doc-gen/
│   ├── discovery/           # Phase 1: repo map + component list
│   └── analysis/            # Phase 2: per-component analysis
└── doc-usage/
    ├── interfaces.md        # Service A's public interface
    └── used-by-<serviceB>/  # How B uses A
```

Add `.ai-doc/` to your `.gitignore` -- these are intermediate working files.

Review notes live at `docs/review-notes.md` alongside the generated docs, where users can see and edit them directly.
