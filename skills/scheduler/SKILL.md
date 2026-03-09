---
name: scheduler
description: Manage scheduled jobs that run claude commands using tmux + cron. Supports recurring and one-time jobs. Use /scheduler add, /scheduler list, /scheduler remove, /scheduler restart, /scheduler logs, /scheduler cleanup.
disable-model-invocation: true
argument-hint: [add "<command>" every N min | add "<command>" at <time> | list | remove <id> | restart <id> | logs <id> | cleanup]
allowed-tools: Bash, Read, Write
---

Manage scheduled Claude Code jobs using tmux sessions + crontab. Each job gets its own tmux session (for auth/Keychain access) and a crontab entry that sends commands into it.

Two job types:
- **Recurring**: runs on an interval (every N min/hour), tmux session stays alive
- **Once**: runs at a specific time, tmux session exits after completion

The user invoked: `/scheduler $ARGUMENTS`

## Setup

Before running any commands, resolve these paths dynamically:

```bash
PROJECT_DIR="$(pwd)"
CLAUDE_BIN="$(which claude)"
TMUX_BIN="$(which tmux)"
JOBS_FILE="${PROJECT_DIR}/jobs.json"
LOGS_DIR="${PROJECT_DIR}/logs"
```

## Operations

Parse the arguments to determine the operation:

### `add "<command>" every <N> <unit>` (recurring)

Create a new recurring job.

1. **Parse arguments**:
   - Extract the command (quoted string or everything before "every")
   - Extract interval: e.g., "every 5 min", "every 1 hour". Default: every 5 min.
   - Convert to cron expression.

2. **Generate job ID**:
   ```bash
   ID="$(date +%Y%m%d)-$(head -c 4 /dev/urandom | xxd -p | head -c 4)"
   ```

3. **Ensure logs directory exists**:
   ```bash
   mkdir -p "${LOGS_DIR}"
   ```

4. **Create tmux session** (from current terminal — has Keychain access):
   ```bash
   ${TMUX_BIN} new-session -d -s "claude-job-${ID}"
   ```

5. **Add crontab entry**:
   - Read current crontab: `crontab -l 2>/dev/null || echo ""`
   - Append:
     ```
     # claude-job:<id> type:recurring
     <cron_expr> <TMUX_BIN> send-keys -t "claude-job-<id>" "CLAUDECODE= <CLAUDE_BIN> -p --dangerously-skip-permissions '<command>' >> <LOGS_DIR>/<id>.log 2>&1" Enter
     ```
   - Write back: pipe full content to `crontab -`

6. **Save to jobs.json**:
   - Read existing jobs.json (create `{"jobs":{}}` if missing)
   - Add entry:
     ```json
     "<id>": {
       "type": "recurring",
       "command": "<command>",
       "cron": "*/N * * * *",
       "interval_human": "every N min",
       "tmux_session": "claude-job-<id>",
       "log_file": "logs/<id>.log",
       "project_dir": "<PROJECT_DIR>",
       "claude_bin": "<CLAUDE_BIN>",
       "tmux_bin": "<TMUX_BIN>",
       "created": "<ISO timestamp>"
     }
     ```
   - Write back

7. **Confirm**: Show job ID, command, interval, and how to check logs.

### `add "<command>" at <time>` (one-time)

Create a one-time scheduled job.

1. **Parse arguments**:
   - Extract the command (quoted string or everything before "at")
   - Extract time: e.g., "at 5pm", "at 3:30pm", "at 14:00", "at 5pm tomorrow"
   - Convert to a pinned cron expression:
     - "at 5pm" (today) → `0 17 <today_day> <today_month> *`
     - "at 3:30pm" → `30 15 <today_day> <today_month> *`
     - "at 5pm tomorrow" → `0 17 <tomorrow_day> <tomorrow_month> *`
   - If the time has already passed today and no "tomorrow" specified, schedule for tomorrow.

2. **Generate job ID, ensure logs dir** (same as recurring)

3. **Create tmux session** (same as recurring):
   ```bash
   ${TMUX_BIN} new-session -d -s "claude-job-${ID}"
   ```

4. **Add crontab entry** — note the `; exit` at the end to close tmux after completion:
   - Read current crontab: `crontab -l 2>/dev/null || echo ""`
   - Append:
     ```
     # claude-job:<id> type:once
     <pinned_cron_expr> <TMUX_BIN> send-keys -t "claude-job-<id>" "CLAUDECODE= <CLAUDE_BIN> -p --dangerously-skip-permissions '<command>' >> <LOGS_DIR>/<id>.log 2>&1; exit" Enter
     ```
   - Write back: pipe full content to `crontab -`

5. **Save to jobs.json**:
   ```json
   "<id>": {
     "type": "once",
     "command": "<command>",
     "cron": "0 17 7 3 *",
     "scheduled_for": "2026-03-07T17:00:00",
     "tmux_session": "claude-job-<id>",
     "log_file": "logs/<id>.log",
     "project_dir": "<PROJECT_DIR>",
     "claude_bin": "<CLAUDE_BIN>",
     "tmux_bin": "<TMUX_BIN>",
     "created": "<ISO timestamp>"
   }
   ```

6. **Confirm**: Show job ID, command, scheduled time, and note that tmux will auto-exit after completion.

### `list` (or no arguments)

1. Read `jobs.json`
2. For each job, check:
   - Tmux session alive: `${TMUX_BIN} has-session -t "claude-job-<id>" 2>/dev/null`
   - Crontab entry exists: `crontab -l 2>/dev/null | grep "claude-job:<id>"`
3. Determine status:
   - **Recurring**: tmux alive + cron exists → `active` | tmux dead + cron exists → `stopped (tmux dead)` | neither → `orphaned`
   - **Once**: tmux alive + cron exists → `pending` | tmux dead + cron exists → `completed` | neither → `orphaned`
4. Display table:
   ```
   | ID | Type | Command | Schedule | Status |
   | 20260307-a3f2 | recurring | /monitor-slack #general | every 5 min | active |
   | 20260307-b7c1 | once | check PR status | at 5pm (Mar 7) | completed |
   ```
5. If there are completed one-time jobs, suggest: "Run `/scheduler cleanup` to remove completed one-time jobs."

### `remove <id>`

1. Kill tmux session: `${TMUX_BIN} kill-session -t "claude-job-<id>" 2>/dev/null`
2. Remove crontab entry:
   - Read crontab
   - Remove the comment line `# claude-job:<id>` AND the command line below it
   - Write back
3. Remove from jobs.json
4. Confirm

### `restart <id>`

1. Read job config from jobs.json (use stored paths for claude_bin and tmux_bin)
2. Kill existing tmux session if present: `${TMUX_BIN} kill-session -t "claude-job-<id>" 2>/dev/null`
3. Create new tmux session: `${TMUX_BIN} new-session -d -s "claude-job-<id>"`
4. Verify crontab entry still exists, re-add if missing
5. Confirm

### `logs <id>`

1. Show the last 50 lines of `${LOGS_DIR}/<id>.log`

### `cleanup`

Remove all completed one-time jobs and orphaned entries.

1. Read `jobs.json`
2. For each job:
   - If type is `once` and tmux session is dead → it has completed
   - If type is any and both tmux and cron are gone → orphaned
3. For each job to clean up:
   - Kill tmux session if somehow still alive
   - Remove crontab entry if still present
   - Remove from jobs.json
4. Show what was cleaned up

## Important notes

- Tmux sessions MUST be created from the user's terminal (not from cron) to inherit Keychain/auth access
- Each job gets its own tmux session to avoid command overlap
- The `send-keys` approach means cron types the command into the tmux session's shell
- For one-time jobs, `; exit` after the command closes the tmux session — the stale cron entry is harmless (send-keys to a dead session is a no-op)
- If a previous command is still running when cron fires again, the send-keys will queue — this is generally fine for short-lived `claude -p` calls
- When listing jobs, always cross-check tmux and crontab state against jobs.json
- All paths (claude, tmux, project dir) are resolved dynamically and stored in jobs.json so the skill is portable
