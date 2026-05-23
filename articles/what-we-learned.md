# What We Learned

This was the root cause of the confusing behavior:

Antigravity and Antigravity IDE can use the same AI service but still keep
separate local application state.

On macOS, the two apps had separate directories:

```text
$HOME/.gemini/antigravity
$HOME/.gemini/antigravity-ide
```

and separate VS Code application support folders:

```text
$HOME/Library/Application Support/Antigravity
$HOME/Library/Application Support/Antigravity IDE
```

## The visible sidebar is an index

The conversation files were not the whole story. The visible list of
conversations was backed by SQLite state in:

```text
User/globalStorage/state.vscdb
```

The key that mattered most was:

```text
antigravityUnifiedStateSync.trajectorySummaries
```

If the IDE has raw conversation files but does not have the matching summary
index, it may not show those conversations.

## File migration and UI migration are different

The file migration moves things like:

```text
brain
conversations
browser_recordings
html_artifacts
context_state
knowledge
prompting
scratch
annotations
implicit
```

The UI migration updates `state.vscdb`.

Both are needed for a complete user-facing migration.

## Avoid one-way destructive changes

The safe pattern is:

1. close both apps
2. back up source and destination
3. merge instead of overwrite when possible
4. verify SQLite integrity
5. reopen the IDE only after the filesystem and DB changes are complete

This matters because the destination IDE may already contain new conversations.
Overwriting the destination index can hide or lose that new state.

## What success looked like

After migration, Antigravity IDE could see the old conversation history from:

```text
$HOME/.gemini/antigravity
```

to:

```text
$HOME/.gemini/antigravity-ide
```

The result was one practical working state: Antigravity IDE became the primary
local app, while existing history remained available.
