# AWS-0004: Worktrunk — bounded adoption for worktree entry and inspection

Status: **Accepted with restrictions.** Decided 2026-11-03 in a1web/engineering-standards#23 after a real pull-request trial (a1web/rodeo#460).

## Rule in one line

**Worktrunk helps us enter and inspect worktrees. The Cora Worktree Toolkit governs how they leave.**

## Approved Worktrunk uses

| Use | Command | Notes |
|---|---|---|
| Discover and inspect worktrees | `wt list`, `wt list --format=json` | Works across worktrees created by any tool |
| Enter or create a worktree | `wt switch <branch>`, `wt switch --create <purpose>/<name>` | Must land at the canonical path (see "Paths") |
| Check out a pull request | `wt switch pr:<n>` | Optional acceleration; tracks `origin/<head-branch>` |
| Seed caches into a new worktree | `wt step copy-ignored --require-include` | Optional; only with an explicit, reviewed `.worktreeinclude` (see "Cache seeding") |
| Status sweep before retirement | `wt list` | The `⊂` marker is pre-retire evidence only (see "Retirement") |

Worktrunk does not validate the toolkit's purpose allowlist. Branch names must still use an approved purpose (`feature`, `feat`, `fix`, `hotfix`, `chore`, `docs`, `test`, or the repository's configured `cora.purpose` values).

## Prohibited Worktrunk uses

Do not use Worktrunk to remove, prune, merge, or delete anything:

- `wt step prune`
- `wt remove`
- `wt merge` (and the `wt step` building blocks `commit`, `squash`, `rebase`, `push`, `promote`, `relocate` as a substitute for the reviewed PR workflow)
- branch deletion of any kind
- worktree retirement or abandonment
- creating or owning safety refs
- deciding final merged-state acceptance
- any destructive lifecycle automation, including Worktrunk hooks that remove, prune, or delete

**Do not rely on `remove.delete-branch = false` for safety.** In the trial, `wt step prune --dry-run --min-age 0s` reported that it would remove both the worktree and the branch with that setting in effect. It would also have deleted ~100 MiB of copied ignored caches without a specific warning. Separately, `wt remove` deleted a worktree with ignored files and one with a running process inside, both of which the toolkit refuses. No Worktrunk setting is part of our safety model.

## Lifecycle authority

The Cora Worktree Toolkit (`bin/worktree`, `bin/repo`) is the only approved path for:

- worktree retirement (`retire --disposition accepted --evidence pr:<n>`)
- abandonment (`retire --disposition abandoned --evidence ledger:<file>` with an owner-signed ledger entry)
- merged-head verification: the PR must be MERGED in the verified origin, with head branch equal to the worktree branch and head SHA equal to the worktree HEAD
- branch preservation: retirement keeps the branch and writes `refs/cora/retired/<branch>/<operation-id>`
- receipts and evidence capture
- governed cleanup and refusal (dirty, unpushed, locked, occupied, or ignored-file-bearing worktrees)

**Branch deletion** is not performed by Worktrunk or the toolkit. The toolkit never deletes branches. Deleting a branch is an explicit, owner-authorized action outside both tools. Remote head branches are deleted only through the repository's GitHub merge settings.

## Retirement

1. **Status sweep (optional):** `wt list` marks a branch fully contained in the target branch with `⊂`. Treat this as a hint that the worktree is ready to retire, **not** as authority that retirement is safe.
2. **Clear seeded caches** if cache seeding was used (see below).
3. **Retire with the toolkit:** `bin/worktree retire <owner/repo> <purpose/name> --disposition accepted --evidence pr:<n>`. Use `--dry-run` first if unsure. The toolkit independently verifies the merged PR and head before removing anything.

## Cache seeding

Copied caches are disposable acceleration material. Their only purpose is to make a new worktree warm faster.

- Use `wt step copy-ignored --require-include` with a `.worktreeinclude` that lists only build and dependency caches (for example `node_modules/`, `public/vite/`, `tmp/cache/`). Without `--require-include`, Worktrunk copies **every** ignored file, including `.env*` files and other credentials. Never run it without that flag.
- Never include credentials, `.env*`, keys, databases, logs, uploads, or runtime state.
- Before retirement, remove the seeded caches from **that worktree only**:

  ```bash
  cd <worktree>
  git clean -n -d -X    # review the list first
  git clean -f -d -X    # then remove
  ```

  `-d` is required: `git clean -f -X` without it does not remove ignored directories.

> **Warning:** `git clean -f -d -X` deletes **every** ignored file in the worktree. Run it only in a worktree whose ignored contents are known to be seeded caches, with nothing intentionally preserved. It is not a general repository cleanup command. Never run it in a default-branch worktree, a repository root, or a worktree you did not seed. If anything in the dry-run list is unexpected, stop.

A helper that wraps this step is deferred until needed. If one is built, it must list what it will remove first, act only inside the target worktree, and fail closed on any uncertainty.

## Paths

Keep the repository-local, purpose-oriented layout. Do not use a central `.worktrees` directory or `~/Projects/OSS`.

```text
~/Projects/<owner>/<repo>/.bare            # shared object database
~/Projects/<owner>/<repo>/<default-branch>/
~/Projects/<owner>/<repo>/<purpose>/<name>/
```

- A1Web repositories: `~/Projects/a1web/<repo>`, for example `~/Projects/a1web/rodeo/feature/some-change`
- Repositories owned by `scarver2`: `~/Projects/scarver2/<repo>`, for example `~/Projects/scarver2/howdy-world`

Configure Worktrunk so its paths match this layout (user config `~/.config/worktrunk/config.toml`):

```toml
worktree-path = "{{ repo_path }}/../{{ branch }}"
```

For a bare layout, `{{ repo_path }}` is the `.bare` directory, so a branch `feature/x` lands at `<repo>/feature/x`.

## Evidence

Trial on the Mac Studio, Worktrunk `wt v0.81.0` (Homebrew), toolkit V1.4 `b0a5fda5`:

- **Checkout:** `wt switch pr:460` checked out a1web/rodeo#460 at the canonical path in 14.1 s, including fetch.
- **Cache seeding:** `copy-ignored --require-include` cloned 84 MiB of caches in 0.6 s with no extra disk (APFS clone).
- **Merge:** the PR merged at head `884cb1d` as merge commit `e7c8744`. The Apple Silicon lockfile fix stayed clean after `bundle check`, `bundle exec`, and a Rails/Vite boot.
- **Status:** after merge, `wt list` marked the worktree `⊂`.
- **Prune:** `wt step prune --dry-run --min-age 0s` would have removed the worktree and branch despite `remove.delete-branch = false`.
- **Cleanup:** `git clean -f -X` left the ignored directories; `git clean -f -d -X` removed them.
- **Retirement:** `retire --disposition accepted --evidence pr:460` verified the merged head, kept the branch, wrote safety ref `refs/cora/retired/chore/add-arm64-darwin-platform/20261103T213416648Z-f55b0159`, and wrote a receipt.

Full record: a1web/engineering-standards#23.
