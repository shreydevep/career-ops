# Fork workflow

This checkout is a fork of [`career-ops-hq/career-ops`](https://github.com/career-ops-hq/career-ops)
for feature work of our own.

```
origin    https://github.com/shreydevep/career-ops.git      (our fork)
upstream  https://github.com/career-ops-hq/career-ops.git   (canonical)
```

| Branch | Purpose | State |
|---|---|---|
| `custom` | what we run and develop on | based on release `career-ops-v1.39.0` |
| `main` | mirror of upstream `main`, never commit here | tracks `origin/main`, currently 110 commits ahead of the release |

`main` is ahead of `custom` because it carries unreleased upstream work.
`custom` stays on the release we actually run until we choose to move.

## Everyday loop

```bash
git switch custom
# ... edit ...
npm run ui                     # web UI on http://127.0.0.1:3000 (dev)
node doctor.mjs                # environment still sane
node test-all.mjs              # full suite (slow)
git add -A && git commit -m "feat: ..." && git push
```

Opening a PR to upstream is possible from here: `gh pr create --repo career-ops-hq/career-ops --head shreydevep:custom`.

## Staying current with upstream

```bash
git fetch upstream --tags
git merge upstream/main        # or: git rebase career-ops-v1.40.0
```

Conflicts are expected in any file we touched. Resolve them in the open; that is
the point of a fork.

## The one trap: don't run the auto-updater on `custom`

`node update-system.mjs apply` does **not** merge. It fetches from
`career-ops-hq` directly (the `origin` remote is irrelevant to it) and then
checks out upstream's version of each system path with `git checkout <sha> -- <path>`.
On `custom` that silently overwrites our changes in the working tree. Our commits
stay in history and `rollback` can restore them, but the working files are gone.

So pick one:

- **Manual sync** (recommended here): `git merge upstream/main`, keep the auto-updater unused.
- **Auto-updater, then reapply**: run `apply` on the `main` worktree only, and rebase `custom` onto the new commit.

`node update-system.mjs check` is read-only and safe to run anywhere; it reports
release versions, not branches.

## Where customizations belong

Ordered by how little friction they cause on the next upstream merge:

1. **User layer** — outside this checkout, in `~/career-data` (CV, profile, portals, tracker, reports). Never code, never touched by updates.
2. **`plugins/`** — the documented extension point; a plugin's code is ours alone.
3. **`modes/_custom.md`, `config/`** — user-layer overrides read automatically by every mode.
4. **`templates/` packs** — new `cover-letter-template.<name>.html` / CV template variants selected by config, instead of editing the shipped ones.
5. **Core files** — only when the above cannot express it, and expect merge conflicts there forever.

## Privacy

**This fork is public**, because a fork of a public repository cannot be private.
No personal data is tracked here: `cv.md`, `portals.yml`, `data/`, `reports/`,
`jds/`, `output/` and `.env` are gitignored, and the real copies live in
`~/career-data`, which has its own private repository
([`shreydevep/career-data`](https://github.com/shreydevep/career-data)).
Keep it that way: never `git add -f` a file from the data root into this tree.
