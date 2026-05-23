# Mac Migration Guide

This article explains the migration path from Antigravity to Antigravity IDE on
macOS. It is based on a successful migration where old Antigravity conversations
were made visible in Antigravity IDE.

## What needs to move

There are two independent parts.

The file backing store:

```text
$HOME/.gemini/antigravity
$HOME/.gemini/antigravity-ide
```

The UI/index state:

```text
$HOME/Library/Application Support/Antigravity/User/globalStorage/state.vscdb
$HOME/Library/Application Support/Antigravity IDE/User/globalStorage/state.vscdb
```

The most important SQLite key we found was:

```text
antigravityUnifiedStateSync.trajectorySummaries
```

This key contains the trajectory summaries used by the app UI to list
conversations. Another useful key is:

```text
antigravityUnifiedStateSync.sidebarWorkspaces
```

That one carries sidebar workspace state.

## Why symlinking only `brain` is not enough

The `brain` directory contains conversation files and generated metadata. But
Antigravity IDE also relies on `state.vscdb` to know which conversations exist,
their titles, timestamps, and sidebar placement.

If `brain` is copied or symlinked without the SQLite state, the files can be
present while the UI still looks empty.

## Safe migration sequence

Quit both apps first:

```sh
osascript -e 'quit app "Antigravity"'
osascript -e 'quit app "Antigravity IDE"'
```

Check that the apps are no longer running:

```sh
pgrep -fl 'Antigravity|Antigravity IDE|language_server'
```

Create backups:

```sh
mkdir -p "$HOME/antigravity-migration-backup"

cp "$HOME/Library/Application Support/Antigravity/User/globalStorage/state.vscdb" \
  "$HOME/antigravity-migration-backup/Antigravity-state.vscdb"

cp "$HOME/Library/Application Support/Antigravity IDE/User/globalStorage/state.vscdb" \
  "$HOME/antigravity-migration-backup/Antigravity-IDE-state.vscdb"

ditto "$HOME/.gemini/antigravity" \
  "$HOME/antigravity-migration-backup/gemini-antigravity"

ditto "$HOME/.gemini/antigravity-ide" \
  "$HOME/antigravity-migration-backup/gemini-antigravity-ide"
```

Verify databases before and after changes:

```sh
sqlite3 "$HOME/Library/Application Support/Antigravity/User/globalStorage/state.vscdb" \
  "pragma integrity_check;"

sqlite3 "$HOME/Library/Application Support/Antigravity IDE/User/globalStorage/state.vscdb" \
  "pragma integrity_check;"
```

## Practical migration approach

For our migration, we used a migrator that handled both:

1. copying the `~/.gemini/antigravity` backing data into the IDE backing store
2. merging the relevant SQLite keys from old Antigravity into Antigravity IDE

The important detail is that `trajectorySummaries` is not plain JSON in the
database. In our setup it was stored as encoded protobuf-like data. A naive SQL
`REPLACE` can overwrite destination state and lose newer IDE conversations.

The safer strategy is:

1. decode old and new summary values
2. merge them
3. write the merged value back
4. keep backups so rollback is possible

## Verification checklist

After migration:

```sh
sqlite3 "$HOME/Library/Application Support/Antigravity IDE/User/globalStorage/state.vscdb" \
  "pragma integrity_check;"
```

Count backing-store conversations:

```sh
find "$HOME/.gemini/antigravity-ide/brain" -mindepth 1 -maxdepth 1 -type d | wc -l
find "$HOME/.gemini/antigravity-ide/conversations" -maxdepth 1 -type f -name '*.pb' | wc -l
```

Then reopen Antigravity IDE and confirm the old conversations appear in the UI.

## Rollback

Quit both apps, then restore:

```sh
ditto "$HOME/antigravity-migration-backup/gemini-antigravity-ide" \
  "$HOME/.gemini/antigravity-ide"

cp "$HOME/antigravity-migration-backup/Antigravity-IDE-state.vscdb" \
  "$HOME/Library/Application Support/Antigravity IDE/User/globalStorage/state.vscdb"
```

Reopen Antigravity IDE after restoring.

