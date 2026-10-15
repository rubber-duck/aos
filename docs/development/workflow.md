# Development workflow

[Agent guide](../../AGENTS.md) · [Coordination](coordination.md) · [Task template](task-template.md) · [Handoff template](handoff-template.md)

## Read, scope, and claim

Beads owns executable work. Inspect architecture and dependencies before claiming or creating a task. Use a unique `BEADS_ACTOR` for the session; shared Git authorship does not identify a claim owner.

```sh
export BEADS_ACTOR="aos-<name>-<unique-session>"
bd prime
bd ready --json
bd show <task-id> --json
bd update <task-id> --claim --json
```

Verify claim success and ownership before editing. Create missing work using the [task template](task-template.md), and keep task bodies in temporary files when using multiline input.

```sh
bd create "Concrete outcome" --type task --body-file <task-body-file> --acceptance "Observable completion criteria" --json
bd dep add <dependent-task> <prerequisite-task>
```

The first task waits for the second. Parent epics group work; explicit blocking edges control readiness. Review proposal tasks against the current architecture before implementation.

## Implement and coordinate

Record the branch/worktree, integration base, scope, and integration owner. Concurrent implementations use separate source worktrees and share the primary tracker. Claims establish responsibility, not file locks; coordinate overlapping files and shared interfaces.

Name the smallest environment that can observe the change and the inputs that would invalidate its evidence. Finish coherent changes before routine checks. Use independent work while a dependency or check is pending, without changing a running check's inputs. Inspect failures before rerunning and keep full logs out of routine progress messages.

## Validate

For the current documentation-only repository, review relative links, code fences, design consistency, and `git diff --check`. For staged changes, also run `git diff --cached --check`. Beads setup changes require a successful connection, dependency/readiness inspection, and recovery from published history when sync changes.

No runtime test suite or language-specific formatter exists yet. Establish maintained build, format, test, and CI entry points with the selected stack. Documentation checks must not be presented as proof of actor behavior.

Future behavior changes include focused tests plus maintained scenarios using real participating actors, hosts, transports, resource services, and inspector interfaces where affected. Use predictable fixtures, await observable readiness and completion, assert outcomes and relevant failure semantics, retain failure evidence, and clean up owned participants. Unit tests supplement integration evidence. Do not build every deployment arrangement for every edit; expand coverage when the changed boundary requires it.

Before an authorized push or merge, validate the combined change and affected callers using the established project gate. One integration owner owns combined evidence. Do not invent IPP commands or require an unrelated full test matrix. Reuse unchanged passing evidence and report skipped or unavailable coverage accurately.

## Review and handoff

Review the actual diff, contract changes, and validation evidence. Passing checks do not approve an unresolved architectural decision. Update owning docs in the same change as accepted decisions and behavior.

| Status | Meaning |
| --- | --- |
| `open` | Unclaimed work |
| `in_progress` | Claimed and active |
| `blocked` | A concrete unmet input or dependency prevents progress |
| `in_review` | Review or integration remains |
| `closed` | Acceptance passed and work is integrated or explicitly accepted |

Use the [handoff template](handoff-template.md) in Beads notes. Closing unblocks consumers, so the required result must exist on their integration base. Checkpoint and sync task history through the designated sync owner; report a failed push as local-only. Clean up task worktrees after integration while preserving recoverable work.

This workflow adapts the general coordination and review practices of IPP. It does not adopt IPP's runtime architecture or implementation tools.
