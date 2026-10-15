# AOS — Actor OS

AOS explores a small actor runtime for autonomous, multimodal agents: hierarchical messaging, permission-aware communication, inspectable state, causal audit histories, and cooperative suspension and resumption. JSON messages reference static attachments and live binary streams carried through separate data channels.

This repository starts with a working system-design brief and project coordination. No runtime, SDK, inspector, implementation language, or application stack has been selected or implemented. Proposed mechanisms and unresolved guarantees remain explicitly open.

- [Architecture and original design brief](docs/architecture.md)
- [Implementation approach](docs/plans/README.md)
- [Agent guide](AGENTS.md)
- [Development workflow](docs/development/workflow.md)
- [Beads setup and recovery](docs/development/coordination.md)

## Working on AOS

Read the architecture before choosing implementation work. Beads owns tasks, dependencies, claims, acceptance criteria, and handoffs; the documents own design and contributor guidance. After following the coordination setup, use `bd ready` to find unblocked work and `bd show <task-id>` to inspect its scope.

The first design work establishes initial environments and the host/SDK/actor boundary, followed by the minimum protocol, delivery, lifecycle, authority, resource, and audit contracts. Native subprocesses and browser execution are design directions; WebSocket is a candidate transport. These are not yet implementation commitments.
