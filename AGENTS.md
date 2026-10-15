# AOS — Agent Guide

AOS is an actor runtime for autonomous, multimodal agents. Read the [working architecture](docs/architecture.md) and verify actual capabilities before assuming a proposal is implemented. The application stack is undecided; do not import IPP's language, build, rendering, or tooling choices.

## Start every task

1. Inspect `git status --short` and `git worktree list`. Preserve existing edits and deletions.
2. Read the architecture, relevant strategy in `docs/plans/`, this guide, and any more specific `AGENTS.md` in paths you will edit.
3. Read the [workflow](docs/development/workflow.md). Establish scope and validation from the user's request and state the immediate approach briefly.
4. Start or join Beads using the [coordination guide](docs/development/coordination.md), set a unique `BEADS_ACTOR`, inspect dependencies, and successfully claim a task before implementation.

After compaction or handoff, reread the relevant current documents and inspect the actual diff and task state. A previous summary is not proof of completion.

## Source of truth

| Location | Owns |
| --- | --- |
| `docs/architecture.md` and future `docs/architecture/` topics | System boundaries, ownership, invariants, accepted decisions, and explicitly open design questions |
| `docs/plans/` | Directional implementation approaches and validation strategies that follow the architecture |
| Source and adjacent module documentation | Concrete APIs, algorithms, formats, and maintained examples once implementation exists |
| Beads | Executable work, milestones, dependencies, claims, acceptance criteria, and handoffs |
| This guide and `docs/development/` | Contributor workflow, review, coordination, and validation |

Review inherited tasks against current architecture. A task does not override the design or authorize unrelated work. Keep live status, checkboxes, task queues, and detailed acceptance criteria in Beads; do not create a parallel `TODO.md`, `MEMORY.md`, or roadmap tracker. Link to the owning document instead of duplicating decisions.

The initial architecture is the supplied working brief, not a claim that its open decisions are settled. Before adopting a new foundational decision or changing an accepted one, prepare a concrete proposal with behavior, rationale, and tradeoffs for user review. An explicit decision already approved in the conversation needs no repeated approval. Record accepted decisions in architecture, then align strategy, tasks, and implementation. Continue independent work while a decision awaits review. Routine implementation choices within accepted architecture and editorial fixes do not need another design review.

## AOS design guardrails

- Keep control messages and bulk/live binary transfer separate. JSON Schema validation does not enforce authority or resource access.
- Keep namespace, supervision, cancellation, and child lifetime distinct until their relationship is specified.
- Distinguish acceptance from completion, inspectable state from checkpoint state, and host-observed facts from actor-reported outcomes.
- Do not claim exactly-once external effects, deterministic replay, rollback, crash durability, zero-copy transfer, or preservation of an absent live stream without an explicit supported contract and evidence.
- Preserve causal relationships and uncertainty in audit records. A restored checkpoint cannot undo an external effect.
- Treat routing permission and OS-level actor isolation as separate concerns. Do not silently add cluster consensus, replication, automatic placement, or transparent migration as requirements.

## Working rules

- Use `rg` for discovery and inspect relevant code before changing it. Batch independent reads and keep changes bounded.
- Complete authorized reversible work without repeated permission requests. Tracker operations do not independently authorize source publication or destructive actions.
- Use a separate branch/worktree for concurrent implementation, with an explicit integration owner for shared interfaces. Manual worktrees belong under ignored `.worktrees/<task-id>/<purpose>/` in the primary checkout. Linked worktrees share the primary Beads database; never initialize another tracker there.
- Preserve other owners' work. Exchange coherent snapshots on a known base; uncommitted files do not follow a new worktree. Report changed contracts and blockers promptly; coalesce routine progress updates.
- Finish a coherent edit, then run the smallest check that answers a concrete correctness question. Broaden checks for affected boundaries and callers. Reuse passing evidence while relevant inputs remain unchanged.
- Keep README files focused on purpose, boundaries, and useful entry points. Keep Markdown paragraphs on single source lines; preserve code-block layout. Establish language and formatter conventions when the stack is chosen.
- Use plain descriptive commit messages and PR titles without type prefixes. Stage explicit paths and review the staged diff. Do not add AI attribution, co-author trailers, or telemetry.

## Validate and hand off

Follow the [workflow](docs/development/workflow.md) and record actual validation in Beads. Documentation-only changes need link, content, and whitespace review, not runtime tests. Future behavior changes need focused tests and maintained scenarios exercising the real participants and affected transports. A build or mock-only test does not establish runtime integration behavior.

Record owner, branch/worktree/base, changed behavior or decisions, exact checks and results, omitted checks, blockers, next action, and last task-history sync result. Use `in_review` while review or integration remains. Close only after acceptance criteria pass and the result is integrated or explicitly accepted.

After integration, stop task-owned processes and remove task worktrees after preserving any remaining unmerged or uncommitted work. Keep the primary checkout. Verify `git worktree list` and record cleanup in Beads.

`CLAUDE.md` points to this guide; edit `AGENTS.md` to keep both entry points consistent.
