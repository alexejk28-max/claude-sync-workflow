# sync-workflow — a git safety net for Claude Code

A skill plus two slash commands that make Claude Code sync a project with GitHub
**only when you ask it to**, and that keep several machines working on the same
repository without stepping on each other.

## Why

Most projects are no longer tied to one computer. Three days in the office, two at home,
sometimes a laptop in between — the work has to follow you, in full, without a "wait, I
left that on the other machine" moment on Monday morning.

Git already solves the hard part. What breaks in practice is the routine around it: you
forget to pull before you start, you leave uncommitted work behind on Friday, and the
first twenty minutes of every session go into figuring out which clone is ahead of which.

This turns that routine into two commands, identical on every machine:

```
/sync_pull     when you sit down
/sync_push     when you get up
```

The agent shows you what changed, asks once, and does the rest — no branch juggling, no
half-remembered git incantations, and a clear answer when two machines did touch the
same files.

The second half of the problem is the agent itself. Coding agents like to be helpful:
left alone, they commit and push whenever a piece of work looks finished — everything
staged, on whatever branch happened to be checked out, with a message nobody wants in
the history. Here, nothing reaches the remote without you seeing the file list first and
saying yes. Two steps, both gated by your confirmation.

## What you get

| File | Purpose |
|------|---------|
| `skills/sync-workflow/SKILL.md` | The rules: procedures, error and conflict handling, security, device sync, checklist |
| `commands/sync_push.md` | `/sync_push` — show status, ask, commit, push |
| `commands/sync_pull.md` | `/sync_pull` — fetch, ask, pull, report the consequences |
| `GUIDE.md` | Step-by-step install, configuration, usage, and troubleshooting |

Core guarantees:

- No `git add/commit/push/pull` without an explicit request from you.
- Status and diff are always shown, and confirmed, **before** anything is staged.
  Untracked files are listed by name — "commit everything" never means files you have
  not seen.
- The staged diff is scanned for credential-shaped files and content before the commit.
- No `--force`, no history rewriting, no `git add -f` around `.gitignore`.
- A pull conflict is never "resolved" with `checkout --` or `reset` — the agent stops
  and asks.
- After a pull, the agent tells you whether you need to restart the session or reinstall
  dependencies — and shows you the diff of anything under `.claude/`, hooks, or MCP
  manifests, because those run code at session start.
- Text that arrives through a pull is treated as data. A skill, README, or commit
  message from the remote does not get to authorize actions.

## Installation

Short version below; step-by-step walkthrough with examples and troubleshooting in
[GUIDE.md](GUIDE.md).

Copy the files into your Claude Code configuration.

**Per project** (recommended — the settings are project-specific anyway):

```bash
cp -r skills/sync-workflow  <your-project>/.claude/skills/
cp commands/sync_push.md commands/sync_pull.md  <your-project>/.claude/commands/
```

**Globally**, for all projects: use `~/.claude/skills/` and `~/.claude/commands/`
instead. Note that the configuration table then has to fit every project.

## Configuration

Open `skills/sync-workflow/SKILL.md` and fill in the table under **Configuration**:

| Setting | What to put there |
|---------|-------------------|
| Repository | `owner/repo` |
| Local path | The project root |
| Branch | Your default branch |
| Commit format | The commit message shape you want |
| Restart-relevant paths | Paths whose changes require restarting the agent session — long-running servers, MCP server source, agent config |
| Dependency manifests | `package.json`, `requirements.txt`, `go.mod`, … |

Everything else in the skill is deliberately generic. The commands need no editing
unless you want a different commit message format — change it in both places.

## Usage

```
/sync_push
```

Shows what changed, asks what to include, commits, pushes, reports the hash.

```
/sync_pull
```

Fetches, shows the incoming commits, asks, pulls, and tells you what the incoming
changes mean for your running session.

## Working on several devices

All clones are equal peers; there is no "main machine". `/sync_pull` when you sit down
at a device, `/sync_push` when you get up. Machine-specific things — absolute paths,
launch scripts, local credentials — stay out of the repository via `.gitignore`.

If a push is rejected as non-fast-forward, another device pushed first: `/sync_pull`,
then push again. The agent will not force anything.

## Notes

- Written for Claude Code, but the files are plain Markdown — any agent that reads
  skills and slash commands from a directory can use them.
- The shell blocks use plain `git` commands and work on Linux, macOS, and Windows alike.
- The skill assumes the project directory *is* the repository. If you sync through a
  mirror or a deploy step, adapt the procedures before using it.

## License

MIT — see [LICENSE](LICENSE).
