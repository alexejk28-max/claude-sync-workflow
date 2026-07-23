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
**List the untracked ones (`??`) separately and by name** — those are the files nobody
has reviewed yet, and the usual way a `.env` or a key file reaches a public repository.
No changes: report "nothing to push" and stop.

### 2. Get confirmation

> "These files will be committed and pushed: [list].
> New/untracked among them: [list]. All of them — or only some?"

Wait for the answer. "All" or no restriction: `git add -A`.
Specific files: add only those.

### 3. Stage and check

```bash
git add <selection>
git diff --cached --stat
```

Scan the staged set before committing: `.env`, `*.pem`, `*.key`, `id_rsa`,
`credentials.json`, `.npmrc`, `.netrc`, dumps — and `API_KEY=`, `SECRET`, `TOKEN`,
`PASSWORD`, `BEGIN PRIVATE KEY` inside the diff. On a hit: stop and ask.

### 4. Commit

Message format: `Update <files/topic> - YYYY-MM-DD HH:MM`
Example: `Update config.json, analyze.md - 2026-07-07 14:30`

```bash
git commit -F -
```

Pass the message on stdin or as a single quoted argument. Do not build a shell string
out of arbitrary text — backticks and `$(...)` in a commit message are command
substitution.

### 5. Push

```bash
git push origin main
```

On non-fast-forward: STOP — do not force. Show `git log HEAD..origin/main --oneline`
and ask (another device has probably pushed first, so run `/sync_pull`).

### 6. Confirmation

Short summary: files, commit hash, push result.
