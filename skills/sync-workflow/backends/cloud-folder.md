# Backend: cloud-folder (git over Drive / Dropbox / OneDrive)

Use this when the user wants real version control — history, branches, merging — but no
GitHub account. A cloud folder (Google Drive, Dropbox, OneDrive, Nextcloud, …) becomes
nothing more than the transport. **git still does all the work**, so every rule in the
main `SKILL.md` applies unchanged: status/diff, confirmation, secret checks, conflict
handling. The only thing that differs is where the remote lives.

## The one rule that makes this safe

Put a **bare** repository inside the cloud folder. Keep the **working tree outside** it.

A bare repo (`*.git`, no working files) is only ever written during `push`/`fetch`, in
short bursts. A normal working tree with a live `.git/` is written constantly, and a
cloud client syncing it mid-write **will corrupt the repository** — this is the single
most common way people lose data with this setup. Never point a synced folder at a
working tree.

```
~/Drive/repos/my-project.git      <- bare repo, this is the "remote"  (inside the cloud folder)
~/projects/my-project             <- working tree, where you actually edit  (OUTSIDE the folder)
```

## One-time setup (first machine)

```bash
# 1. Create the bare repo inside the cloud folder
git init --bare "~/Drive/repos/my-project.git"

# 2. In your existing working tree, point origin at it (a plain filesystem path)
cd ~/projects/my-project
git remote add origin "~/Drive/repos/my-project.git"

# 3. First push
git push -u origin main
```

Wait until the cloud client reports the `.git` bare repo as fully uploaded before
touching another machine.

## One-time setup (each additional machine)

```bash
# Let the cloud client finish downloading ~/Drive/repos/my-project.git first, then:
git clone "~/Drive/repos/my-project.git" ~/projects/my-project
```

Record the bare-repo path as **Repository** in the skill's configuration table, and set
**Sync backend** to `cloud-folder`.

## Daily use

`/sync_push` and `/sync_pull` work exactly as in `SKILL.md` — `origin` just happens to
be a local path instead of a URL. The one habit to add:

- **Before `/sync_push`:** make sure the cloud client has finished **downloading** any
  pending remote changes, so your push builds on the latest bare repo.
- **After `/sync_push`:** wait for the client to finish **uploading** before starting
  work on another machine. Pushing while an upload is mid-flight is how the two machines
  diverge.

Non-fast-forward on push means another machine pushed first, same as with GitHub:
`/sync_pull`, then push again.

## If the bare repo gets corrupted anyway

Symptoms: `fatal: bad object`, `unable to read tree`, push/fetch errors that name
objects. Usually the cause is a working tree that ended up in the synced folder, or two
machines writing the bare repo at once.

- **Do not** try to repair it in place while the cloud client keeps syncing — pause the
  client first.
- Every clone is a full copy of the history. The safe recovery is to build a fresh bare
  repo from a healthy clone and push into it:

  ```bash
  # from a machine whose working tree is intact
  git init --bare "~/Drive/repos/my-project-new.git"
  cd ~/projects/my-project
  git remote set-url origin "~/Drive/repos/my-project-new.git"
  git push --all origin
  git push --tags origin
  ```

  Then repoint the other machines' `origin` at the new bare repo. Do this only with the
  user's confirmation, and never delete the old bare repo until the new one is verified.

## Maximum-caution variant: git bundle

If even a bare repo in the synced folder feels risky, sync a single `git bundle` file
instead — one self-contained file the cloud client can never catch mid-write:

```bash
# push equivalent: write a bundle into the cloud folder
git bundle create "~/Drive/repos/my-project.bundle" --all

# pull equivalent on another machine: fetch from the bundle file
git fetch "~/Drive/repos/my-project.bundle" 'refs/heads/*:refs/remotes/bundle/*'
```

Slower and more manual, but a bundle is atomic from the client's point of view. Offer it
only if the user asks for extra safety; the bare repo is fine for normal use.
