# Implementation approach

[Architecture](../architecture.md) owns the design, including unresolved choices. This document describes how to investigate and validate it. Beads owns the sequence, dependencies, milestones, and acceptance criteria. None of the proposed implementation work selects a language or stack.

## Contract exploration

Resolve the initial execution environments and host/SDK/actor responsibilities before committing to APIs or persistence mechanisms. Specify the minimum message envelope, identity, acceptance, lifecycle, observation, resource, and authorization contracts together. Walk through normal delivery, a lost acknowledgment and duplicate, suspension with pending work, a disconnected stream, and an uncertain external outcome. Record decisions in their owning architecture topic and retain unresolved alternatives explicitly.

## Local runtime and inspection

Use predictable parent and child test actors with a real router and the selected execution backend. Observe authorized and denied sends, acceptance versus completion, inspectable state, and causally linked audit events. Keep reusable scenarios independent of process launch details so additional drivers can exercise another backend later. An initial inspector should make the selected contracts and failure outcomes observable.

## Checkpoints and resources

Exercise cooperative suspension with pending work, actor recreation, and resumption through the selected checkpoint contract. Test static attachment access and retention separately from live-stream buffering, fan-out, disconnect, and recovery. Fixtures should expose loss or uncertainty rather than implying that checkpointing preserves a live source or reverses external effects. Capture the relevant host observations and actor reports when failures occur.

## Federation and browser participation

Once local contracts are demonstrated, exercise a mounted remote subtree across a real connection, including identity, authority, reconnect, and cross-host resource access. A browser participant must account for outbound-only WebSocket connection establishment and browser lifecycle restrictions. Use an audio-device/model scenario to observe media behavior and measure delay and queue growth during concurrent bulk transfer. Select transport and environment coverage from the accepted contracts; do not infer latency guarantees before measurement.
