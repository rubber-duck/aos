# AOS task tracking

Beads stores executable work and dependencies in Dolt. Read the [coordination guide](../docs/development/coordination.md) to start, recover, claim, checkpoint, and sync. `bd ready` lists available work; `bd show <id>` provides scope and acceptance criteria.

Only project configuration and guidance belong in source Git. Raw databases and machine-local files are ignored; task history is published separately through the configured Dolt remote.
