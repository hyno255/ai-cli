---
name: reflect
description: Self-improvement skill. Interviews the user about a failed skill run, actively reproduces the issue, learns the correct approach, and proposes a targeted fix to skill/agent definitions.
disable-model-invocation: true
argument-hint: [skill-name]
---

# Self-Improvement / Reflection Orchestrator

You are orchestrating a self-improvement loop. A skill or agent produced poor output, and the user knows what went wrong. Your job: interview the user, **actively reproduce the failure**, learn what the correct approach is, and propose a minimal fix to the skill/agent definitions.

**Arguments:**
- `$ARGUMENTS[0]` = **optional** name of the skill that failed (e.g. `doc-gen`)

---

## Phase 1 — Interview

**Goal:** Understand what went wrong and how the user discovered the correct answer.

1. If `$ARGUMENTS[0]` is provided, use it as the target skill. Otherwise, use `AskUserQuestion` to ask which skill or agent produced the bad output.
2. Ask the user what happened — what was the expected output vs. what the skill actually produced?
3. Follow up naturally based on their answers. Don't follow a rigid script. Examples of useful follow-ups:
   - "How did you find the correct answer?"
   - "What commands or queries did you run?"
   - "What tools/logs/dashboards did you check?"
   - "What did the output look like when you found it?"
   - "Was there a specific step the agent missed or got wrong?"
4. Keep asking until you have enough context to attempt reproduction yourself. You need:
   - The target skill name and the specific failure
   - What the user did differently to get the right result
   - Any commands, queries, or tools the user used

**When done:** Save the interview summary to `.ai-doc/reflect/interview.md` with sections:
- **Target skill**: name and file path
- **Failure**: what went wrong
- **User's approach**: how they found the correct answer
- **Key steps to reproduce**: the commands/queries/actions to try

---

## Phase 2 — Active Exploration & Reproduction

**Goal:** Independently reproduce the failure and the fix. Learn by doing, not just reading.

This is the core of the skill. You must actively try things, not just read files and guess.

### Step 1 — Read the target skill's files

Read the target skill's `SKILL.md` and any referenced agents, guides, templates, or checklists. Understand what the skill currently instructs the agent to do.

### Step 2 — Try to reproduce

Follow the user's described steps. Run the same commands or queries. Check the same tools or logs. See if you get the same results the user described.

### Step 3 — Compare

- What did the skill instructions tell the agent to do?
- What did the user actually do to find the right answer?
- Where is the gap? Be specific — point to the exact instruction (or missing instruction) in the skill files.

### Step 4 — Ask for more context if needed

If reproduction fails or results differ from what the user described, ask for clarification using `AskUserQuestion`. Don't guess — ask.

### Step 5 — Draft a hypothesis

Propose a specific fix idea. E.g.:
> "The researcher agent doesn't check Argus logs. Adding an instruction to query Argus for `<pattern>` in the discovery phase should surface the missing information."

### Step 6 — Test the hypothesis

Run the proposed steps yourself to verify they produce the correct result. If you proposed adding a command to the skill, run that command now and check its output.

### Step 7 — Iterate

If the test fails, ask the user for more guidance. **Max 3 exploration loops** through Steps 2–6. If after 3 loops you still can't reproduce, summarize what you tried and what's still unclear, then proceed to Phase 3 with your best hypothesis.

**When done:** Write the diagnosis to `.ai-doc/reflect/diagnosis.md`:
- **Root cause**: What specific instruction or omission in the skill caused the failure
- **Evidence**: The gap between what the skill does vs. what the user did
- **Reproduction steps**: What you ran and what you observed
- **Proposed fix**: Concrete, minimal changes to skill/agent files

---

## Phase 3 — Apply Fix

**Goal:** Present and apply a targeted fix, while also surfacing observations that help the skill grow over time.

1. Read `.ai-doc/reflect/diagnosis.md` for the proposed changes.
2. **Triage the skill's current instructions** in light of what you learned during reproduction. Categorize your observations:
   - **Keep (working well):** Instructions that contributed to correct behavior or are essential. Call these out so they don't get accidentally removed in future edits.
   - **Add (missing):** New instructions or steps needed to fix the reported issue. This is the core fix.
   - **Remove or revise (harmful/misleading):** Existing instructions that actively caused the failure, produce misleading results, or are outdated. These should be removed or rewritten — not just worked around by adding new instructions on top.
   - **Watch (uncertain):** Instructions you suspect may cause problems but couldn't confirm during reproduction. Log these for future runs to investigate.
3. Present the proposed changes to the user as before/after diffs. For each change, explain **why** it fixes the issue, referencing your reproduction evidence from Phase 2. Include your keep/add/remove/watch triage so the user sees the full picture.
4. Wait for user approval via `AskUserQuestion`:
   - "Apply all changes"
   - "Apply with modifications" (then ask what to change)
   - "Skip — don't apply"
5. Apply only approved edits. Keep changes **minimal** — fix the reported issue and nothing else. No refactoring, no style changes, no "while we're here" improvements.
6. Append a changelog entry to `.ai-doc/reflect/changelog.md`:

```markdown
## <date> — <target-skill>

**Issue:** <one-line summary>
**Root cause:** <one-line summary>
**Fix:** <list of files changed and what changed>
**Evidence:** <how reproduction confirmed the fix>
**Triage notes:**
- Keep: <instructions confirmed as working well>
- Remove/revise: <instructions that were harmful or misleading>
- Watch: <instructions to monitor in future runs>
```

The changelog accumulates across runs. Before proposing a fix, check it for past entries on the same skill to avoid regressions.

---

## Phase 4 — Validate (optional)

**Goal:** Verify the fix actually resolves the issue.

1. Ask the user if they can provide a reproducible test case or want to re-run the skill now.
2. If yes:
   - Re-run the fixed skill (or the relevant portion) with the same input that originally failed
   - Check if the output now matches expectations
   - If not resolved: loop back to Phase 2 (**max 2 validation retries**)
   - If resolved: report success
3. If no test case available: skip validation and remind the user to watch for the issue on the next real run.

---

## Done

Summarize what happened:
- The issue that was reported
- The root cause identified
- The fix applied (or proposed, if user declined)
- Whether validation passed (if attempted)

Remind the user that past fixes are logged in `.ai-doc/reflect/changelog.md` and will be checked in future runs to prevent regressions.
