# Remote Sync Service Follow-up

After the app migration, we had a local service that still read archived
conversation data from the old Antigravity folder.

The old assumption was:

```text
$HOME/.gemini/antigravity
```

After moving to Antigravity IDE, the service needed to read:

```text
$HOME/.gemini/antigravity-ide
```

## Better than hard-coding

Instead of replacing one hard-coded path with another, we moved the local service
to a config file.

Example:

```json
{
  "artifactRoot": "$HOME/.gemini/antigravity-ide",
  "antigravityBin": "$HOME/.antigravity-ide/antigravity-ide/bin/antigravity-ide"
}
```

The important lesson is that app-local storage can move again. A config file
makes the next move a restart and a one-line edit, not another code change.

## Verification

The service was checked by calling its local state endpoint and confirming that
archived conversations were now returned with IDE paths:

```text
~/.gemini/antigravity-ide/brain/...
```

Artifact reads were also checked:

1. reads inside the IDE backing store succeeded
2. reads from the old backing store were blocked by the safety boundary

That confirmed the remote service had switched its trusted artifact root.

