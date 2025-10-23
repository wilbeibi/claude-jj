# Jujutsu Auto-Tracking for Claude Code

## Purpose
Automatically create discrete jujutsu changes for each Claude Code task, making every change easily reviewable and revertable.

## Core Behavior

### Hook Detection
When you see `🪝 JJ_PRE_EDIT` or `🪝 JJ_STOP` in bash tool output, follow this skill.

### Session State
Track internally (in conversation context) whether you've initialized jj for this task:
- `jj_initialized = false` at start
- `jj_initialized = true` after creating change

---

## Implementation

### Step 1: Before First Edit (PRE_EDIT Hook)

When about to edit files for the first time in a conversation:

1. Check if in jj repo:
```bash
jj status
```
If this fails, skip jj integration (not a jj repo).

2. If `jj_initialized == false`, create new change:
```bash
jj new -m "Claude Code: <brief task description>"
```

Extract task description from user's request (first 60 chars, no newlines).

Examples:
- User: "add authentication system" → `jj new -m "Claude Code: add authentication system"`
- User: "fix bug in parser" → `jj new -m "Claude Code: fix bug in parser"`
- User: "refactor the API" → `jj new -m "Claude Code: refactor the API"`

3. Set `jj_initialized = true` in your working memory.

**Critical**: Only do this ONCE per task, before the very first file edit.

---

### Step 2: During Editing

All file edits automatically amend the current change. No action needed.

---

### Step 3: After Task Complete (STOP Hook)

Before giving your final response, show the user what was created:
```bash
jj log -r @ --color=always
jj diff --stat
```

Then include in your response:
```
I've completed your request. Changes tracked in jujutsu:

<show jj log output>

Files changed:
<show jj diff --stat output>

Review options:
- jj diff      - See full changes
- jj commit    - Accept changes  
- jj abandon @ - Revert everything
```

---

## Multi-Round Conversations

**Refinement (same task)**:
```
User: "add auth"
You: [create change, edit files, show results]

User: "change the hash algorithm"  
You: [just edit files - auto-amends same change, show updated results]
```

**New task (different work)**:
```
User: "add auth"
You: [create change, edit files, show results]

User: "now add tests"
You: [run: jj commit to snapshot previous]
     [run: jj new -m "Claude Code: add tests"]
     [set jj_initialized = true]
     [edit test files, show results]
```

**Decision rule**: 
- Refining/fixing same feature → keep editing (auto-amends)
- Distinct new feature/task → commit + new change

---

## Error Handling

If any jj command fails:
- Continue with normal workflow
- Don't block on jj errors  
- Don't mention jj tracking failed

---

## Critical Rules

1. **Initialize once** - Only create change before first edit of a task
2. **Never ask permission** - Just create the change automatically
3. **Always show results** - At end, always show jj log + diff
4. **Fail gracefully** - If jj unavailable, continue normally
5. **Track state** - Remember if you've initialized this conversation

---

## Examples

### Example 1: Simple Task
```
User: "add user authentication"

Internal: jj_initialized = false, about to edit files
Execute: jj new -m "Claude Code: add user authentication"
Internal: jj_initialized = true
Execute: [edit files - auto-amend]
Execute: jj log -r @ && jj diff --stat

Response:
"I've added user authentication with JWT tokens.

Jujutsu change:
@ qpvuntsm "Claude Code: add user authentication"
│ 2 minutes ago

Files changed:
  src/auth.py    | +156
  tests/test.py  | +89

- jj diff - Review
- jj commit - Accept
- jj abandon @ - Revert"
```

### Example 2: Refinement
```
User: "add auth"
You: [create change, work, show results]

User: "use bcrypt instead"
Internal: jj_initialized = true, same task
Execute: [just edit files - auto-amends]
Execute: jj log -r @ && jj diff --stat

Response:
"I've updated to use bcrypt for password hashing.

<show updated change>

The same change now includes bcrypt implementation."
```

### Example 3: Sequential Tasks
```
User: "add login endpoint"
You: [create "login endpoint" change, work, show results]

User: "now add logout endpoint"
Internal: New distinct task
Execute: jj commit
Execute: jj new -m "Claude Code: add logout endpoint"
Internal: jj_initialized = true
Execute: [edit files, show results]

Response:
"I've added the logout endpoint.

<show new change>

This is a separate change from the login work."
```

### Example 4: User Wants Revert
```
User: "actually, revert all those changes"

Execute: jj abandon @

Response:
"Reverted all changes from the previous change.
Your working directory is now clean."
```

