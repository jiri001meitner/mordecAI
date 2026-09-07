# MORDECAI Protocol

**Status:** Draft  
**Protocol generation:** v0  
**Stability:** Experimental

This document defines the initial protocol model for communication between MORDECAI components.

It is intentionally implementation-neutral. Concrete protobuf, CBOR, JSON, QUIC stream, or other encodings may evolve during the PoC.

The first objective is to establish stable semantics before freezing wire bytes.

---

## 1. Goals

The protocol must support:

- heterogeneous workers;
- capability discovery;
- model/object distribution;
- task placement;
- distributed model stages;
- streaming tensors and results;
- failure detection;
- retries;
- useful-work receipts;
- protocol evolution;
- privacy extensions.

The protocol must assume that peers can be unavailable, outdated, malformed, or malicious.

---

## 2. Non-goals for v0

Protocol v0 does not attempt to solve:

- Byzantine consensus;
- fully decentralized scheduling;
- transferable-token or blockchain settlement;
- full MPC transformer inference;
- anonymous credentials;
- global governance;
- arbitrary remote VM execution.

Those may be layered later.

---

## 3. Roles

Initial logical roles:

```text
Client
  │
  ▼
Gateway
  │
  ▼
Orchestrator
  │
  ▼
Scheduler
  │
  ▼
Node
  │
  ▼
Runtime
```

Additional services:

```text
Object Store
Verifier
Accounting
Model Registry
Telemetry
Relay
```

A physical process may implement multiple roles during early development.

Role boundaries remain logical protocol boundaries.

---

## 4. Identifiers

Identifiers should be opaque.

Suggested forms:

```text
node_id
session_id
task_id
attempt_id
object_id
model_id
receipt_id
epoch_id
```

Do not encode sensitive semantics in IDs.

Content-addressed object IDs are an exception because the digest is intentionally semantic to integrity.

Example:

```text
blake3:<hex-digest>
```

---

## 5. Protocol versioning

Every connection handshake must advertise supported protocol versions.

Example semantic structure:

```json
{
  "type": "hello",
  "protocol": {
    "min": 0,
    "max": 0
  }
}
```

Peers select the highest mutually supported version.

If no compatible version exists, terminate cleanly with an explicit error.

Nodes must not assume simultaneous upgrades.

---

## 6. Connection handshake

Conceptual sequence:

```text
Node                                  Control Plane
 │                                         │
 │──────────── ClientHello ───────────────►│
 │                                         │
 │◄─────────── ServerHello ────────────────│
 │                                         │
 │──────── CapabilityAdvertisement ───────►│
 │                                         │
 │◄──────── CapabilityAccepted ────────────│
 │                                         │
 │──────────── Heartbeat ─────────────────►│
```

Authentication and node identity mechanisms are deliberately not frozen in v0.

Transport authentication must nevertheless exist before permissionless operation.

---

## 7. `ClientHello`

Purpose:

- identify protocol compatibility;
- advertise node software version;
- negotiate optional features.

Conceptual schema:

```yaml
type: client_hello
protocol_min: 0
protocol_max: 0
software:
  name: mordecai-node
  version: 0.0.1
features:
  - object_transfer_v0
  - stage_execution_v0
nonce: opaque-random-value
```

Do not use software version as protocol version.

---

## 8. Capability advertisement

Nodes advertise what they can currently provide.

Conceptual schema:

```yaml
type: capability_advertisement
node_id: opaque
sequence: 42

resources:
  scalar_compute:
    architecture: x86_64
    logical_cores: 32

  tensor_compute:
    - backend: cuda
      architecture: sm_90
      memory_bytes: 85899345920
      precision:
        - fp32
        - fp16
        - bf16
        - int8

  bulk_memory:
    bytes: 274877906944

  persistent_storage:
    bytes_available: 1099511627776

network:
  ingress_limit_bytes_per_second: 125000000
  egress_limit_bytes_per_second: 125000000

policy:
  max_concurrent_jobs: 2
```

Capability claims are hints until verified.

Schedulers must not treat self-advertised performance as trusted truth.

---

## 9. Heartbeats and leases

Presence should use leases rather than permanent registration.

A node periodically renews availability.

Example:

```yaml
type: heartbeat
node_id: opaque
sequence: 991
active_attempts: 1
resource_pressure:
  memory: 0.62
  compute: 0.88
```

If the lease expires, the scheduler marks the node unavailable.

Do not immediately destroy historical reputation or identity because a lease expired.

---

## 10. Task model

A task is a logical unit of useful work.

A task may have multiple execution attempts.

```text
Task
 ├── Attempt A → timeout
 ├── Attempt B → result
 └── Attempt C → verification replica
```

Economic accounting must distinguish a task from an attempt.

---

## 11. Task offer

The scheduler offers work to a node.

Conceptual schema:

```yaml
type: task_offer
task_id: opaque
attempt_id: opaque
operation: model_stage_inference

requirements:
  tensor_backend:
    - cuda
    - rocm
  memory_bytes: 21474836480
  precision: bf16

objects:
  - blake3:MODEL_SHARD_DIGEST

deadline_ms: 30000
execution_budget_ms: 15000
privacy_class: standard
```

A task offer does not imply acceptance.

---

## 12. Task acceptance

Node response:

```yaml
type: task_accept
task_id: opaque
attempt_id: opaque
reservation:
  expires_in_ms: 5000
```

or:

```yaml
type: task_reject
task_id: opaque
attempt_id: opaque
reason: insufficient_memory
retryable: true
```

Rejecting work is normal.

Nodes must not be punished merely for accurately rejecting work that violates current local policy or resource availability.

---

## 13. Object protocol

Model shards and other large artifacts are content-addressed.

Object metadata:

```yaml
object_id: blake3:<digest>
size_bytes: 2147483648
media_type: application/x-mordecai-model-shard
encoding: raw
```

Retrieval:

```text
ObjectRequest
    ↓
ObjectMetadata
    ↓
ObjectChunk*
    ↓
ObjectComplete
```

The receiver MUST verify the complete object digest before trusting or executing the object.

Chunk-level hashes may be added for resumability and early corruption detection.

---

## 14. Model manifests

A model manifest maps logical stages to immutable artifacts.

Example:

```yaml
schema_version: 1

model:
  id: core-test
  version: 1

stages:
  - id: stage-0
    object: blake3:...
    estimated_memory_bytes: 18000000000

  - id: stage-1
    object: blake3:...
    estimated_memory_bytes: 19000000000
```

Production manifests must eventually be signed or authenticated.

---

## 15. Stage execution

A distributed inference stage receives tensors and produces tensors.

Conceptual request:

```yaml
type: execute_stage
task_id: opaque
attempt_id: opaque
model:
  manifest: blake3:...
  stage: stage-1

input:
  stream_id: opaque
  dtype: bf16
  shape:
    - 4
    - 2048
    - 4096
```

Tensor payloads should use dedicated high-throughput streams rather than embedding large tensors in control messages.

---

## 16. Tensor streams

Tensor streams require metadata:

```text
stream ID
dtype
shape
encoding
expected byte length
task/attempt binding
integrity information
```

Receivers MUST enforce declared size limits before allocating unbounded memory.

Control messages and tensor payloads should be independently multiplexable.

---

## 17. Streaming inference

For user-facing generation:

```text
Gateway
   │
   ▼
Core / pipeline
   │
   ├──── token/result delta
   ├──── token/result delta
   ├──── token/result delta
   ▼
completion
```

Internal pipeline streams are not equivalent to public API streams.

The gateway owns public stream semantics.

---

## 18. Result messages

Conceptual result:

```yaml
type: task_result
task_id: opaque
attempt_id: opaque
status: success

result:
  object: blake3:...

metrics:
  execution_ms: 812
  bytes_read: 2147483648
  bytes_received: 67108864
  bytes_sent: 67108864
```

Self-reported metrics are not automatically trusted for accounting.

---

## 19. Work receipts

A work receipt records evidence of useful execution.

Conceptual structure:

```yaml
schema_version: 1
receipt_id: opaque
task_id: opaque
attempt_id: opaque
node_id: opaque

work:
  operation: model_stage_inference
  result_digest: blake3:...

verification:
  state: pending

measurement:
  execution_ms: 812

signature: ...
```

Lifecycle:

```text
executed
   ↓
receipt
   ↓
verification / confidence
   ↓
Compute Credit accounting
```

---

## 20. Verification

Verification is operation-specific and is **not a solved universal primitive**.

General trustless verification of heterogeneous neural-network computation remains an open research problem for MORDECAI. A receipt records an execution claim and evidence; it does not by itself prove that arbitrary AI computation was correct.

Early controlled prototypes may use trusted development nodes. Permissionless accounting must progressively introduce workload-specific verification.

Verification is operation-specific.

Possible policies:

```text
none                development only
deterministic       exact recomputation
replicated          compare independent workers
challenge           known/partially known task
statistical         tolerance-based model result
specialized         operation-specific verifier
```

A task should identify the required verification policy.

---

## 21. Errors

Errors must be machine-readable.

Suggested structure:

```yaml
type: error
code: NO_COMPATIBLE_WORKER
message: No compatible worker is currently available.
retryable: true
```

Initial error families:

```text
PROTOCOL_UNSUPPORTED
MESSAGE_INVALID
AUTHENTICATION_FAILED
CAPABILITY_REJECTED
TASK_UNKNOWN
TASK_EXPIRED
TASK_CANCELLED
RESOURCE_UNAVAILABLE
OBJECT_NOT_FOUND
OBJECT_CORRUPT
MODEL_INCOMPATIBLE
EXECUTION_FAILED
TIMEOUT
RATE_LIMITED
INTERNAL_ERROR
```

Public errors must not leak internal peer addresses or sensitive topology.

---

## 22. Cancellation

Cancellation is explicit.

```yaml
type: cancel_attempt
task_id: opaque
attempt_id: opaque
reason: speculative_winner_selected
```

Nodes should stop work promptly where safe.

Compute Credit accounting must define whether partial useful work is creditable.

---

## 23. Idempotency and duplicates

The network may deliver duplicate logical messages due to retries.

Operations that can safely be idempotent should be.

`attempt_id` and sequence values should prevent duplicate crediting or duplicate state transitions.

Exactly-once delivery must not be assumed.

---

## 24. Timeouts

Every remote operation requires a bounded timeout.

Timeout values may be negotiated or assigned by task class.

Use monotonic time for local timeout measurement.

Persisted timestamps use UTC.

---

## 25. Transport

QUIC is the preferred initial candidate for node transport.

Potential stream classes:

```text
control
object transfer
tensor input
tensor output
receipt
telemetry
```

This is not yet a frozen wire requirement.

The implementation must benchmark alternatives rather than treating QUIC as theology.

---

## 26. Security requirements

Before permissionless deployment:

- peers must be authenticated;
- control messages must be integrity protected;
- replay-sensitive messages require replay protection;
- object digests must be verified;
- parser limits must be enforced;
- task authorization must be verified;
- receipts must be cryptographically bound to attempts;
- private keys must never traverse ordinary protocol messages.

---

## 27. Privacy extension points

Protocol messages should permit future replacement of clear semantics with opaque references.

Future extensions may support:

- relay paths;
- hidden worker topology;
- padded frames;
- secret-shared tensor streams;
- MPC cell negotiation;
- anonymous credentials;
- unlinkable accounting receipts.

Protocol v0 should avoid unnecessary fields that make future privacy impossible.

---

## 28. Scheduler/control-plane decentralization

The initial PoC may use a centralized scheduler.

The wire protocol should avoid assuming that exactly one scheduler can ever exist.

Future control-plane models may include:

- regional schedulers;
- federated schedulers;
- elected coordinators;
- distributed placement;
- privacy-preserving rendezvous.

Do not solve this before basic distributed inference works.

---

## 29. Protocol state example

```text
DISCONNECTED
    │
    ▼
CONNECTING
    │
    ▼
NEGOTIATING
    │
    ▼
AUTHENTICATED
    │
    ▼
CAPABILITY_REGISTERED
    │
    ▼
AVAILABLE
    │
    ├──── task offer ───► RESERVED
    │                        │
    │                        ▼
    │                     RUNNING
    │                        │
    │                  ┌─────┴─────┐
    │                  ▼           ▼
    │               COMPLETE     FAILED
    │                  │
    │                  ▼
    │               RECEIPT
    │
    └──── lease expiry ───► UNAVAILABLE
```

---

## 30. First implementation target

Protocol v0 is successful when it can support this experiment:

1. start multiple MORDECAI nodes;
2. discover their capabilities;
3. place different model stages on different nodes;
4. fetch content-addressed model shards;
5. stream activations between stages;
6. produce a streamed inference response;
7. survive one worker disappearing;
8. record useful-work receipts;
9. expose enough metrics to explain performance.

Everything else is secondary until this works.
