# AOS — Actor OS

Working system-design brief · September 22, 2026

AOS is an actor runtime for autonomous, multimodal agents. It combines hierarchical messaging, permission-aware communication, inspectable state, detailed audit histories, and suspension/resumption from serialized state. Static media and live streams travel through separate data channels referenced by ordinary JSON messages.

This document captures the current design direction. Recommendations and unresolved choices are identified explicitly; implementation languages, libraries, and exact guarantees remain to be selected.

## Purpose and core model

AOS is a technical side project exploring how a small runtime can support agents that are understandable, controllable, and composable. An actor can represent an LLM agent, a tool, an audio device, a model connection, an audit collector, or an infrastructure service.

Each host runs a local actor environment. Its core responsibilities are name resolution, message routing and queueing, authentication, and basic permission enforcement. Actors can spawn children and communicate asynchronously using hierarchical addresses such as `/assistant/researcher`.

The original execution model uses subprocesses. Browser-hosted nodes are also a desired direction, so the protocol should accommodate different execution backends. Browser actors would use workers or in-process execution rather than OS subprocesses. The exact division of process lifecycle, scheduling, supervision, and persistence responsibilities between host, SDK, and actors is open.

The guiding preference is to implement services through actor interfaces wherever practical. A gateway, attachment service, or audit collector can be an actor. This does not require bulk media to pass through ordinary actor message queues.

The hierarchy provides a namespace and a way to express scoped access. Whether the same hierarchy also determines supervision, cancellation, and child lifetime must be specified separately.

## Messages, attachments, and streams

The current direction is **JSON messages with references to binary resources**. This supports ordinary JSON Schema for message definitions and avoids embedding images, audio, or video as base64 in routine actor traffic. BSON was explored, but binary payloads no longer require choosing a binary encoding for the message envelope.

An illustrative message—not a finalized schema—might be:

```json
{
  "type": "audio.consume",
  "to": "/assistant/model",
  "resources": [
    {
      "kind": "stream",
      "ref": "microphone-stream",
      "mediaType": "application/octet-stream",
      "format": {
        "encoding": "pcm",
        "sampleRate": 24000,
        "channels": 1,
        "sampleFormat": "s16le"
      }
    }
  ]
}
```

The envelope will also need identity, authorization, and correlation/causation information. Exact field names and versioning rules are open. JSON Schema can describe structure; runtime policy must enforce permissions and resource access.

Two resource types share a reference mechanism but have different behavior:

| Resource | Meaning | Consumer behavior |
|---|---|---|
| Static attachment | Completed binary content, such as an image or recording | Fetch the content; optionally cache or read a portion |
| Live stream | Content produced over time, such as microphone audio | Open or subscribe to a channel and consume incrementally |

The router is the logical entry point for resolving resources and authorizing access. Storage and transfer can be provided by separate services. A reference may resolve to a direct connection or a relayed channel, depending on the environment.

For example, an audio-device actor advertises a stream, and a model actor opens it. Audio then travels through a binary channel while coordination continues through JSON messages. This avoids serializing every audio chunk into JSON. Framing, buffering, copying, and any required model-provider format conversion still exist; direct transfer and zero-copy behavior are not automatic consequences of references.

Recommended resource rules to resolve in the protocol are immutable static content, explicit readiness, access checks, and retention that accounts for pending work, checkpoints, and audit needs. Live streams need separate rules for buffering, slow consumers, multiple subscribers, disconnection, and whether missed content remains available.

## Observation, audit, and resumable state

Every actor should implement standard interfaces for inspection and progress reporting. State-change events should identify their causal inputs so a UI can explain which message or operation led to a change. Observation events are themselves messages and can be consumed by an audit actor or different visualization tools.

The audit goal is detailed historical inspection: what was sent, what the actor reported, what changed, what operations were attempted, and which outcomes were confirmed. Deterministic execution replay and rollback of the external world are not requirements. A distributed timeline should preserve causal relationships and distinguish recorded facts from missing or uncertain outcomes.

Three related facilities need distinct contracts:

- **Inspectable state:** the serializable view an actor exposes to operators and tools.
- **Checkpoint state:** the serialized logical state needed to resume the actor.
- **Audit and progress events:** records describing activity over time, with retention and reliability policies still to be defined.

An actor must support suspension and resumption from serialized state. The proposed implementation model is cooperative suspension: the actor reaches a safe boundary, represents outstanding work in its checkpoint, and releases or records runtime resources as appropriate. Resumption recreates those resources and continues from the saved logical state.

Checkpointing a stream reference does not preserve a live source or the audio produced while the actor was absent. Likewise, restoring a checkpoint does not undo an external effect. The protocol must define how pending messages, in-flight operations, child actors, and resource references relate to a completed suspension.

The host may know that it routed a message without knowing that an actor applied a state change. Audit records should distinguish host-observed events from actor-reported events. Whether audit completeness is guaranteed across crashes—and what happens when the audit service is unavailable—is open.

## Delivery, authority, and federation

Acknowledgment transfers responsibility: once the actor acknowledges a message, the router can stop retrying it. Acknowledgment and successful completion are separate events.

The durability of that acceptance is unresolved. If accepted work must survive a crash, the actor or a supporting SDK/service must durably record responsibility before acknowledgment. If receipt into memory is sufficient, a crash can lose accepted work. Retries before acknowledgment may produce duplicates. No general exactly-once execution or external-effect guarantee has been established.

Authentication and basic authorization belong at the routing boundary. Actors can additionally check whether a requested operation is permitted. The intended model allows scoped authority, inheritance or delegation, and user-approved elevation. Short-lived scoped tokens are a candidate mechanism; token format, revocation, and delegation semantics remain open.

Permission to contact an actor, access its descendants, inspect its state, fetch its resources, and invoke particular operations may need distinct scopes. Authority carried in messages must have trusted provenance. Messaging permission also does not itself provide OS-level isolation for actor code.

A gateway actor exposes another host's available actors as a mounted subtree. It handles the remote connection while presenting a normal actor-facing interface. Connection roles do not determine which actor can initiate a conversation. Discovery, disconnect behavior, path resolution, cross-host resource access, and trust between hosts need explicit contracts.

**WebSocket is the leading transport candidate**, including for initial binary streams. Its modest framing overhead and browser support fit the portability goal. Control traffic, real-time media, and bulk transfers should have enough separation to prevent large transfers from monopolizing latency-sensitive queues.

A browser node can open bidirectional WebSocket connections but cannot expose a listening WebSocket server through the standard browser API. It would participate through outbound links to native nodes or relays. Browser lifecycle restrictions also require reconnect and recovery behavior. Transport adapters should leave room for local IPC and other media transports if needed.

## Decisions needed before implementation

| Decision | What needs to be established |
|---|---|
| Initial execution environments | Native host, browser host, or both in the first milestone; initial language and SDK |
| Host/SDK/actor boundary | Ownership of spawning, supervision, queues, persistence, and resource transfer |
| Delivery guarantees | Meaning of acceptance, retry limits, duplicate handling, and crash durability |
| Lifecycle contract | Safe suspension, pending work, children, cancellation, and checkpoint compatibility |
| Identity and namespace | Actor identity across restarts, path reuse, remote mounts, and naming ownership |
| Authorization and actor trust | Scope inheritance, delegation, elevation, revocation, and any isolation requirements |
| Audit reliability | Required records, permitted gaps, storage failure behavior, and media retention |
| Resource semantics | Static attachment lifetime; stream buffering, fan-out, deadlines, and reconnection |
| Protocol discovery and evolution | Schema publication, supported operations, versioning, and compatibility |

Automatic placement, transparent migration, replicated hosts, and cluster consensus have not been selected as requirements. Planned suspension and recovery after an unexpected crash should be treated as separate guarantees.

## Proposed first implementation sequence

1. **Specify the minimum contracts.** Define the message envelope, actor lifecycle, acceptance, observation events, resource descriptors, and authorization context. Walk through normal delivery, a lost acknowledgment, suspension during work, and a disconnected stream.
2. **Build a local host and inspector.** Demonstrate a parent and child actors exchanging authorized messages, publishing state, and producing a causal audit view. Begin with predictable test actors.
3. **Add checkpoints and resource services.** Demonstrate suspension, process recreation, and resumption; send a static attachment and connect a live binary stream. Make failure and retention behavior visible in the inspector.
4. **Connect native and browser nodes.** Expose a subtree through a gateway, resolve resources across the connection, and run an audio-device/model example. Measure audio delay and queue growth during concurrent bulk transfer.

This sequence is a proposal. The first milestone and durability requirements still need agreement before choosing implementation details.

Useful initial architecture documents would cover runtime boundaries, the JSON message protocol and schemas, attachment/stream operations, lifecycle and delivery guarantees, authorization, and the audit/inspection model.

## Existing approaches to consult

These are design references, not selected dependencies:

- [XState inspection](https://stately.ai/docs/inspection) and [persistence](https://stately.ai/docs/persistence): actor events, state changes, causal inputs, and saved actor state.
- [Erlang/OTP supervision](https://www.erlang.org/doc/system/sup_princ.html): parent/child lifecycle and restart semantics.
- [NATS authorization](https://docs.nats.io/learn/security/authorization): permissions over hierarchical messaging subjects.
- [Dapr Actors](https://docs.dapr.io/developing-applications/building-blocks/actors/actors-overview/): managed actor identity, lifecycle, and persistent state.
- [Claim-check pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/claim-check): messages referencing separately stored payloads.
- [WebSocket protocol](https://www.rfc-editor.org/rfc/rfc6455.html) and [browser API](https://websockets.spec.whatwg.org/): framing, binary transfer, and browser-facing connection semantics.
