# Analysis Guide for Researchers

This guide tells you what to look for and how to structure your output when analyzing a codebase or component.

## What to Look For

### During Discovery (Phase 1)

Scan the repo structure and identify:

1. **Entry points**: `main` files, `index` files, route definitions, CLI commands, serverless handlers
2. **Service boundaries**: separate directories with their own `package.json`, `go.mod`, `Dockerfile`, `Makefile`, or similar
3. **API definitions**: REST routes, GraphQL schemas, gRPC proto files, OpenAPI specs
4. **Data layer**: database migrations, models/entities, ORM configurations, schema files
5. **Configuration**: environment variables, config files, feature flags, secrets references
6. **External integrations**: message queue consumers/producers, third-party API clients, webhook handlers
7. **Build/deploy**: CI/CD configs, Dockerfiles, deployment scripts, infrastructure-as-code
8. **Tests**: test directories, test utilities, fixtures -- these reveal expected behavior

### During Deep Analysis (Phase 2)

For each component, investigate:

1. **Purpose**: What does this component do? Why does it exist? What problem does it solve?
2. **Key interfaces**: What does it export? What can other components call or subscribe to?
3. **Dependencies inbound**: What does this component depend on? (imports, API calls, DB access, config reads)
4. **Dependencies outbound**: What depends on this component? (who imports it, who calls its APIs)
5. **Workflows**: What end-to-end user flows does this component participate in? What role does it play?
6. **Design decisions**: What patterns does it use? (middleware chain, event sourcing, CQRS, repository pattern, etc.) Why were they chosen?
7. **Error handling**: How does it handle failures? (retries, circuit breakers, fallbacks, error propagation)

## How to Identify Workflows

Workflows are end-to-end user-facing flows that span multiple components. To find them:

1. Start from user-facing entry points (API endpoints, UI routes, CLI commands, scheduled jobs)
2. Trace the execution path through the codebase
3. Note each component that participates and what it does
4. Name the workflow clearly: "User Registration", "Order Checkout", "Invoice Generation"

Good workflow indicators:
- Route handlers that call multiple services
- Event chains (publish → subscribe → process)
- Middleware chains with distinct stages
- State machines or step functions
- Background jobs triggered by user actions

## How to Identify Architectural Patterns

Look for these common patterns:

| Pattern | Indicators |
|---------|-----------|
| Microservices | Multiple independent services with their own entry points, separate deploys |
| Monolith | Single entry point, shared database, everything in one deploy |
| Event-driven | Message queues, event buses, publish/subscribe patterns |
| Layered | Clear separation: handlers → services → repositories → data |
| CQRS | Separate read/write models, command/query separation |
| API Gateway | Central routing layer that delegates to backend services |

## Output Format

### Discovery Output (`repo-map.md`)

```markdown
# Repository Map

## Overview
[1-2 paragraphs: what this project is, what it does, who it's for]

## Tech Stack
- **Language**: [e.g., Go 1.21]
- **Framework**: [e.g., Gin, Echo, Django]
- **Database**: [e.g., PostgreSQL via GORM]
- **Message Queue**: [e.g., RabbitMQ]
- **Other**: [notable tools, libraries]

## Architecture Pattern
[Describe the overall pattern: microservices, monolith, etc. with brief justification]

## Directory Structure
[Key directories with 1-line descriptions -- not every file, just the major areas]

## Entry Points
[List main entry points with file paths]

## External Dependencies
[External services, APIs, databases this project connects to]
```

### Discovery Output (`components.md`)

```markdown
# Identified Components

## auth
- **Path**: `src/auth/`
- **Description**: Handles user authentication and authorization via JWT tokens and OAuth2.
- **Complexity**: medium
- **Key files**: `handler.go`, `middleware.go`, `oauth.go`

## billing
- **Path**: `src/billing/`
- **Description**: Payment processing with support for Stripe and PayPal providers.
- **Complexity**: large
- **Key files**: `service.go`, `stripe.go`, `paypal.go`, `webhooks.go`

[... one section per component ...]
```

### Deep Analysis Output (`<component-name>.md`)

```markdown
# Component: [name]

## Purpose
[1-2 sentences: what it does and why it exists]

## Key Interfaces
[Exported APIs, functions, events, data models -- summarized, not exhaustive]

## Dependencies
### Inbound (this component depends on)
- [component/service] -- [why]

### Outbound (depends on this component)
- [component/service] -- [why]

## Workflows
- **[Workflow Name]**: [Role this component plays in the workflow]

## Design Decisions
[Notable patterns, trade-offs, and why they were chosen]

## File References
- `path/to/file.go` -- [brief description]
- `path/to/other.go:45-80` -- [what this section does]
```
