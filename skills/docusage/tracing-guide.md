# Tracing Guide for Cross-Service Usage Analysis

This guide tells you how to find and analyze cross-service interactions.

## How to Find Interactions

### 1. API Client Calls

Search for HTTP clients, gRPC stubs, or SDK usage that targets Service A:

- **HTTP**: Look for `http.Get`, `http.Post`, `fetch`, `axios`, `requests.get`, etc. with URLs containing Service A's hostname or base path
- **gRPC**: Look for generated client stubs importing Service A's proto definitions
- **SDK**: Look for imports of Service A's client library package
- **Config references**: Look for environment variables or config keys containing Service A's name/URL (e.g., `SERVICE_A_URL`, `AUTH_SERVICE_HOST`)

### 2. Event/Message Consumption

Search for Service B subscribing to events published by Service A:

- **Queue consumers**: Look for queue/topic names that match Service A's published events
- **Event handlers**: Look for event type names that originate from Service A
- **Webhook receivers**: Look for endpoints in Service B that Service A calls back to

### 3. Shared Data Access

Search for Service B accessing data owned by Service A:

- **Database**: Look for queries against tables that belong to Service A's domain
- **Cache**: Look for cache keys with Service A's namespace
- **Shared storage**: Look for file/object access to Service A's storage paths

### 4. Configuration Dependencies

Search for Service B's behavior depending on Service A's configuration:

- **Feature flags**: Look for flag checks that gate Service A-related behavior
- **Config files**: Look for configuration sections referencing Service A
- **Environment variables**: Look for env vars that contain Service A's connection details

## How to Determine "Why"

For each interaction you find, ask:

1. **What business action triggers this?** Trace backwards from the call site to find the user action or event that starts the chain.
2. **What would break if this interaction was removed?** This reveals the dependency's importance.
3. **Is this a synchronous dependency (blocking) or asynchronous (fire-and-forget)?** This affects how tightly coupled the services are.
4. **Is there error handling around this interaction?** Look for retries, circuit breakers, fallbacks, or graceful degradation.

## How to Group by Workflow

After finding all interactions, group them by the user-facing workflow they support:

1. Collect all call sites from Service B to Service A
2. For each call site, trace backwards to the entry point (API handler, event handler, scheduled job)
3. Name the workflow based on the user-facing action (e.g., "Checkout Flow", "User Profile Update")
4. Multiple call sites that serve the same user action belong to the same workflow

## Output Format

### Interface Analysis Output (`interfaces.md`)

```markdown
# Service A: Public Interface

## REST API

### POST /api/v1/users
- **Purpose**: Create a new user account
- **Auth**: Requires service-to-service API key
- **Input**: `{ email, name, role }`
- **Output**: `{ id, email, name, created_at }`
- **Errors**: 409 Conflict (email exists), 400 Bad Request (validation)

[... more endpoints ...]

## Events Published

### user.created
- **Channel**: RabbitMQ exchange `user-events`
- **Payload**: `{ user_id, email, created_at }`
- **When**: After successful user creation

[... more events ...]

## Shared Data Models

### users table
- **Owned by**: Service A
- **Key columns**: id, email, role, status
- **Access pattern**: Read-only by consumers via API (not direct DB access)
```

### Usage Analysis Output (`analysis.md`)

```markdown
# How Service B Uses Service A

## Interaction Summary

| What B Calls | Why | Trigger | Workflow |
|-------------|-----|---------|----------|
| POST /api/v1/users | Create user during signup | User submits registration form | User Registration |
| GET /api/v1/users/:id | Validate user exists | Order placement | Checkout Flow |
| Event: user.created | Sync user to local cache | Async event | User Sync |

## Workflow: User Registration

### Trigger
User submits the registration form via POST /api/signup in Service B.

### Interaction
- **What**: Service B calls `POST /api/v1/users` on Service A
- **Where**: `src/handlers/signup.go:45` calls `authClient.CreateUser()`
- **Data sent**: `{ email, name, role: "customer" }`
- **Data received**: `{ id, email, created_at }`
- **Error handling**: If Service A returns 409, Service B shows "email already registered"

### Why
Service A is the system of record for user accounts. Service B delegates user creation to maintain a single source of truth for identity.

[... more workflows ...]
```
