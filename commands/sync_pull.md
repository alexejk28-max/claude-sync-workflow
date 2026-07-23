# Sync Pull — take over changes from GitHub

The project directory IS the repository — no deploy steps. Details and error cases:
skill `sync-workflow`.

## Steps

### 1. Fetch the remote state (without merging)

```bash
git fetch origin main
git log HEAD..origin/main --oneline
git diff HEAD..origin/main --stat
```

No differences: report "already up to date, no pull needed" and stop.

### 2. Get confirmation

> "GitHub has [N] new commits. Files affected: [list]. Take them over?"

Wait for an explicit confirmation.

### 3. Pull

```bash
git pull origin main
```

On "local changes would be overwritten": do NOT discard. Show the affected files and
ask (commit first via `/sync_push`, or stash).

### 4. Check the consequences and report

- Changes to restart-relevant paths (long-running servers, MCP server source, agent
  configuration files): **restart the agent session** — such processes load their code
  only at session start.
- Changes to a dependency manifest: recommend the matching install command.
- Changes to instruction files, skills, or commands: take effect from the next session.

### 5. Confirmation

Short summary: commits taken over, files affected, follow-up steps required.
