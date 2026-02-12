# Workflow: [Workflow Name]

## Why This Interaction Exists

[1-2 sentences: What business need in Service B requires calling Service A?]

## Trigger

- **What starts it**: [user action, event, scheduled job in Service B]
- **Entry point in B**: `see path/to/handler.go:line`

## Sequence

```mermaid
sequenceDiagram
    participant User
    participant ServiceB
    participant ServiceA
    [Sequence diagram showing the interaction flow]
```

## Interaction Details

### Step 1: [action in Service B]
- **What happens**: [description]
- **Code**: `see serviceB/path/to/file.go:45`

### Step 2: [call to Service A]
- **What B calls**: [endpoint/event/data access]
- **Data sent**: [what data flows from B to A]
- **Data received**: [what comes back from A]
- **Code (caller)**: `see serviceB/path/to/client.go:80`

### Step 3: [response handling in Service B]
- **What happens**: [how B uses A's response]
- **Code**: `see serviceB/path/to/handler.go:95`

## Error Handling

- **If Service A is unavailable**: [what does Service B do? retry? fallback? fail?]
- **If Service A returns an error**: [how does B handle specific error codes?]

## Data Flow Summary

| Direction | Data | Format |
|-----------|------|--------|
| B → A | [what is sent] | [JSON, protobuf, etc.] |
| A → B | [what is returned] | [JSON, protobuf, etc.] |

## Related

- [Other workflows involving these services]
- [Service A interface docs]
