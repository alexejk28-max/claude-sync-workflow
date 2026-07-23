---
name: sync-workflow
description: GitHub synchronization — run /sync_push and /sync_pull correctly, handle conflicts and error cases, keep multiple devices in sync. Use for any git, GitHub, or versioning request in this project.
---

# Sync Workflow (GitHub <-> project directory)

**Ground rule:** run `git add/commit/push/pull` ONLY when the user explicitly invokes
`/sync_push` or `/sync_pull`, or asks for it unambiguously. Never sync on your own
initiative — not even when a piece of work is finished. Offer instead.

The project directory IS the repository — no copy steps, no deploy targets, no mirror
folders. Everything (source, config, docs, agent commands and skills) lives in the repo
and travels through a normal git cycle.

## Configuration

Adapt these values to your own setup — they are the only project-specific parts of this
skill:

| Setting | Value |
|---------|-------|
| Repository | `<owner>/<repo>` |
| Local path | `<project-root>` |
| Branch | `main` |
| Commit format | `Update <files/topic> - YYYY-MM-DD HH:MM` |
| Restart-relevant paths | `<paths whose changes require a restart, e.g. MCP server source>` |
| Dependency manifests | `<e.g. package.json, requirements.txt, go.mod>` |

## /sync_push — procedure

1. `git status --short` + `git diff --stat` in the project root; show the list to the user.
2. **Get confirmation:** commit everything, or only selected files?
3. `git add <selection>`, then commit using the configured message format.
4. `git push origin main`.
5. Short summary: files, commit hash, push result.

## /sync_pull — procedure

1. `git fetch origin main`, then `git log HEAD..origin/main --oneline` and
   `git diff HEAD..origin/main --stat`. No differences: report "already up to date", stop.
2. **Get confirmation**, then `git pull origin main`.
3. **Check and report after the pull:**
   - Restart-relevant paths changed: tell the user to restart the agent session
     (long-running servers load their code only at startup).
   - Dependency manifest changed: recommend the matching install command.
   - Agent configuration changed (instructions file, skills, commands): takes effect
     from the next session on.

## Errors and conflicts

- **"Your local changes would be overwritten" on pull:** do NOT discard. Show which
  files are affected; options are to commit first (via `/sync_push`) or to stash
  (`git stash`, pull, `git stash pop`) — the latter only when no conflict in the same
  files is likely.
- **Non-fast-forward on push:** stop. Show `git log origin/main..HEAD` and the opposite
  direction, then ask. No `--force`, no rebase without an explicit request.
- **Merge conflicts:** name the conflicting files and resolve them together with the
  user. Be especially careful with configuration and rule files that other parts of the
  project treat as the source of truth.

## Security

- Never quote the output of `git remote -v` unabridged — a remote URL can carry an
  embedded token. Credentials belong in a credential helper, never in the URL and never
  in the repo.
- Respect `.gitignore` (dependencies, build output, logs, credentials). Never work
  around it with `git add -f`.
- No `push --force`, no history rewriting.

## Device sync

Any number of clones are equal peers — there is no "main machine". Each clone is a
normal repository; `/sync_push` and `/sync_pull` behave identically everywhere.

- **Start of a work session on any device:** `/sync_pull` first, before making changes.
- **End of a work session:** `/sync_push`, so the next device starts from the current state.
- **Machine-specific things** (absolute paths, launch scripts, local credentials,
  hardware-dependent settings) stay out of the repo — put them in `.gitignore` or in a
  local configuration file that is not tracked.
- A push rejected as non-fast-forward usually means another device pushed first: run
  `/sync_pull`, then push again.

One-time setup on a new device: authenticate (for example `gh auth login`), clone the
repository, install dependencies, restore the machine-specific files listed above.

## Checklist

- [ ] The request was explicit (`/sync_push`, `/sync_pull`, or an equivalent instruction)
- [ ] Status/diff shown and confirmation received BEFORE add/commit/push/pull
- [ ] Commit message follows the configured format
- [ ] After pull: restart and dependency-install hints given where applicable
- [ ] No token quoted, no force, no `add -f`

## Common mistakes

1. **Committing "helpfully" after changing files** — violates the ground rule. Always
   offer, never act.
2. **"Resolving" a pull conflict with `checkout --` or `reset`** — discards the user's
   local work. Stop and ask.
3. **Continuing after a pull that touched restart-relevant paths without saying so** —
   the old version keeps running and behavior stops matching the documentation.
4. **Assuming a structure the project does not have** (mirror folders, deploy steps,
   copies in a second directory). If existing documentation describes something like
   that and the repository does not show it, the documentation is stale: report it
   instead of acting on it.
