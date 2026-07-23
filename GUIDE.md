# Guide — install, configure, use

A short walkthrough from zero to a working `/sync_push` and `/sync_pull`.
Five minutes, most of it copy-paste.

## 1. Install

Claude Code reads skills and slash commands from a `.claude` directory. Put the files
either in the project you want to sync (recommended) or in your home directory (applies
everywhere).

**Per project:**

```bash
mkdir -p <your-project>/.claude/skills <your-project>/.claude/commands
cp -r skills/sync-workflow <your-project>/.claude/skills/
cp commands/sync_push.md commands/sync_pull.md <your-project>/.claude/commands/
```

**Globally:**

```bash
mkdir -p ~/.claude/skills ~/.claude/commands
cp -r skills/sync-workflow ~/.claude/skills/
cp commands/sync_push.md commands/sync_pull.md ~/.claude/commands/
```

On Windows the same paths exist as `%USERPROFILE%\.claude\` — copy with Explorer or
PowerShell, the content is identical.

Restart your Claude Code session afterwards. Skills and commands are read at session
start, so a running session will not see the new files.

Check that it worked: type `/` and look for `sync_push` and `sync_pull` in the list.

## 2. Configure

Open `.claude/skills/sync-workflow/SKILL.md` and fill in the **Configuration** table.
Only this table is project-specific — the rest of the skill works unchanged.

A filled-in example:

| Setting | Value |
|---------|-------|
| Repository | `janedoe/research-notes` |
| Local path | `~/projects/research-notes` |
| Branch | `main` |
| Commit format | `Update <files/topic> - YYYY-MM-DD HH:MM` |
| Restart-relevant paths | `mcp-server/src/`, `.mcp.json` |
| Dependency manifests | `package.json` |

**Restart-relevant paths** is the one worth thinking about. List anything that a running
agent session has already loaded into memory and will not reload on its own: MCP server
source, long-running background services, agent configuration. After a pull that touches
one of these, the agent will tell you to restart instead of letting you debug behavior
that no longer matches the code.

**Dependency manifests** work the same way: if a pull changes one, the agent recommends
the install command instead of leaving you with a broken tree.

If you want a different commit message format, change it in the skill **and** in
`commands/sync_push.md` — both mention it.

### Optional: state the ground rule in your project instructions

The skill enforces "no git without an explicit request" on its own, but it is worth
repeating it in your `CLAUDE.md` so it applies even when the skill is not loaded:

```markdown
Git only on explicit request — `/sync_push` or `/sync_pull`. Never commit or push
automatically, not even when a task is finished. Offer instead.
```

## 3. Use

### Pushing

```
/sync_push
```

The agent shows `git status --short` and `git diff --stat`, then asks what to include.
Answer "all" or name specific files. It commits with the configured message format,
pushes, and reports the commit hash.

Nothing changed? It says so and stops — no empty commits.

### Pulling

```
/sync_pull
```

The agent fetches, lists the incoming commits and the files they touch, and asks before
merging. After the pull it tells you what the changes mean for your session: restart
needed, dependencies to install, or configuration that takes effect next session.

Already up to date? It says so and stops.

### What it will not do

- Commit or push because a task looks finished. It offers; you decide.
- `git push --force`, rebase without being asked, or rewrite history.
- Resolve a pull conflict with `git checkout --` or `git reset` — it stops and shows you
  the affected files instead.
- Bypass `.gitignore` with `git add -f`, or quote a remote URL that may contain a token.

## 4. Several devices

Every clone is an equal peer. The rhythm is:

1. Sit down at a device: `/sync_pull` **before** making changes.
2. Work.
3. Get up: `/sync_push`.

Set up a new device once: authenticate (`gh auth login` or an SSH key), clone, install
dependencies, and recreate the machine-specific files that `.gitignore` keeps out of the
repository — absolute paths, launch scripts, local credentials.

Keep machine-specific settings in a file that is not tracked, for example
`.claude/settings.local.json`, so two devices never fight over it.

## 5. Troubleshooting

**The commands do not appear after `/`.** The files are in the wrong place or the
session is stale. Verify `.claude/commands/sync_push.md` exists, then restart the
session.

**"Your local changes would be overwritten by merge".** You have uncommitted work that
the incoming commits also touch. Either `/sync_push` first, or stash:

```bash
git stash
git pull origin main
git stash pop
```

Stashing is fine when the incoming changes touch other files, and risky when they touch
the same ones — in that case commit first.

**"Updates were rejected because the tip of your current branch is behind".** Another
device pushed first. Run `/sync_pull`, then push again. Do not force.

**Merge conflict.** Git marks the conflicting files; the agent names them and helps
resolve them. Be careful with configuration and rule files that other parts of the
project treat as the source of truth — a silently mismerged config is worse than a
visible conflict.

**The agent committed without asking.** Some other instruction outranked the skill.
Check whether your `CLAUDE.md` or a global instruction file tells it to commit when work
is done, and remove that.
