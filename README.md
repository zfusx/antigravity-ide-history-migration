# Antigravity to Antigravity IDE History Migration

Notes from a real macOS migration from Google Antigravity to Antigravity IDE.

The short version: the visible chat history is not only the files under `~/.gemini`.
Antigravity also stores the conversation sidebar index in the VS Code-style
SQLite state database. A complete migration has to preserve both.

This repository is documentation only. It is not an official Google project.

## Articles

- [Mac migration guide](articles/macos-migration-guide.md)
- [What we learned](articles/what-we-learned.md)

## Main idea

Antigravity and Antigravity IDE use separate local state locations:

```text
$HOME/.gemini/antigravity
$HOME/.gemini/antigravity-ide

$HOME/Library/Application Support/Antigravity
$HOME/Library/Application Support/Antigravity IDE
```

The `~/.gemini/...` folders hold conversation backing data such as `brain`,
`conversations`, browser recordings, artifacts, context state, and knowledge.

The `Application Support/.../User/globalStorage/state.vscdb` database holds
VS Code-style UI state, including the sidebar trajectory summary index.

If you only copy or symlink the `brain` folder, the IDE may have the files but
still not show the conversations in the sidebar.

## Safety

Before touching anything:

1. Fully quit both apps.
2. Back up both `~/.gemini` folders.
3. Back up both `state.vscdb` files.
4. Verify the destination database with `sqlite3 ... "pragma integrity_check;"`.

Do not publish your real `state.vscdb`, `brain`, `conversations`, tokens,
logs, screenshots, or private project paths.
