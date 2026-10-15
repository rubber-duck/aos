# Beads coordination

[Agent guide](../../AGENTS.md) · [Workflow](workflow.md)

## Storage and tools

Beads owns tasks and dependencies in a project-local Dolt database shared by linked source worktrees. Source Git tracks configuration and guidance. Database files, locks, credentials, exports, and per-machine connection state are ignored.

The initial setup uses Beads `1.2.2` and Dolt `2.2.0`, matching the installed, verified pair on the development host. These are coordination tools only; they do not select the AOS application stack. Install matching binaries from the official [Beads releases](https://github.com/gastownhall/beads/releases) and [Dolt releases](https://github.com/dolthub/dolt/releases), verifying release checksums. Review and test recovery before upgrading the pair.

The managed SQL server listens on loopback on a project-specific port. Explicit startup is required. Per-command Dolt commits, automatic pushes, and automatic JSONL export are disabled. `no-git-ops: true` requests explicit source Git management; inspect Git history after initialization or remote configuration, because this Beads version can still commit those setup changes. Routine task writes remain in the SQL working set until the sync owner checkpoints them.

## Existing checkout

```sh
bd dolt start
bd where
bd dolt test
bd prime
bd ready
```

Set a unique `BEADS_ACTOR` and follow the claim procedure in the workflow. Linked worktrees discover the primary `.beads`; do not run `bd init` in each worktree. Independent clones do not share live claims. Coordinate assignments across machines or use a deliberately configured shared server before concurrent work.

## Fresh clone

```sh
git clone https://github.com/rubber-duck/aos.git
cd aos
bd dolt start
bd bootstrap --dry-run
bd bootstrap --yes
bd config set status.custom in_review
bd dolt test
bd ready
```

Inspect the dry run before proceeding: it must recover existing AOS history from the configured remote, not create an empty tracker. The database-local custom status needs reapplying after bootstrap with this tool pair. Verify existing issues and dependencies before writes. Preserve the project `AGENTS.md` and `.beads/PRIME.md` guidance.

## Checkpoint and publish

The initial host uses the existing GitHub CLI HTTPS credential helper for authenticated writes. New contributors need their own GitHub credentials and repository write access to publish.

One owner coordinates checkpoints and sync; a solo session is the sync owner. Do not pull while another owner is writing. The configured remote is `git+https://github.com/rubber-duck/aos.git`.

```sh
bd dolt commit --message "Checkpoint progress and handoffs"
bd dolt pull
bd dolt push
```

Pull after local checkpointing when the remote already exists, review conflicts with the relevant owners, and then push. The initial publication uses push to create the remote history. Beads history is stored in `refs/dolt/data`, separately from source `main`; a source Git push alone does not publish task updates. Source commits and pushes remain explicit actions within the user's authorized scope. See the upstream [sync model](https://github.com/gastownhall/beads/blob/main/docs/core-concepts/sync-concepts.md).

A JSONL export is an interchange/viewer format, not the canonical tracker or a complete recovery copy. Before an upgrade, checkpoint and create a Dolt backup with `bd backup init <destination>` and `bd backup sync`. Preserve schema-version checks. Do not force-push, reinitialize, or discard history to resolve a sync error; preserve the local database and report the failure.
