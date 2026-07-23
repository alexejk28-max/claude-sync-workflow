# Sync Push — push local changes to GitHub

The project directory IS the repository — no copy steps. Details and error cases: skill
`sync-workflow`.

## Steps

### 1. Show the current state

```bash
git status --short
git diff --stat
```

Print a clear list of all modified, new, and deleted files.
No changes: report "nothing to push" and stop.

### 2. Get confirmation

> "These files will be committed and pushed: [list]. All of them — or only some?"

Wait for the answer. "All" or no restriction: `git add -A`.
Specific files: add only those.

### 3. Commit

Message format: `Update <files/topic> - YYYY-MM-DD HH:MM`
Example: `Update config.json, analyze.md - 2026-07-07 14:30`

```bash
git add <selection>
git commit -m "<message>"
```

### 4. Push

```bash
git push origin main
```

On non-fast-forward: STOP — do not force. Show `git log HEAD..origin/main --oneline`
and ask (another device has probably pushed first, so run `/sync_pull`).

### 5. Confirmation

Short summary: files, commit hash, push result.
