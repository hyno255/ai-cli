# Workflow: [Workflow Name]

## Why This Exists

[1-2 sentences: What business need or user action triggers this workflow? Why is it important?]

## Overview

[Brief summary of what happens end-to-end in this workflow]

## Trigger

- **What starts it**: [user action, scheduled event, external webhook, etc.]
- **Entry point**: [file path and function/handler that receives the trigger]

## Sequence

```mermaid
sequenceDiagram
    [Sequence diagram showing the key steps and components involved]
```

## Steps

### 1. [Step Name]
- **Component**: [which component handles this step]
- **What happens**: [brief description]
- **Why**: [business reason for this step]
- **Key code**: `see path/to/file.go:45-80`

### 2. [Step Name]
- **Component**: [which component handles this step]
- **What happens**: [brief description]
- **Why**: [business reason]
- **Key code**: `see path/to/file.go:90-120`

[... additional steps ...]

## Data Flow

[Describe what data moves between components during this workflow. What is the input? What is the output? What transformations happen along the way?]

## Error Handling

[How does this workflow handle failures? Retries? Rollbacks? User-facing error messages?]

## Related

- [Link to related workflows]
- [Link to relevant API docs]
