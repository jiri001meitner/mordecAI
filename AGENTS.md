# AGENTS.md

# MORDECAI — Agent Engineering Guide

This file defines the operating rules for AI coding agents and automated contributors working in the MORDECAI repository.

Read this file before modifying the repository.

Also read:

- [`README.md`](README.md) — project purpose and public overview
- [`ARCHITECTURE.md`](ARCHITECTURE.md) — system architecture and design direction

If implementation details conflict with the architectural invariants in `ARCHITECTURE.md`, do not silently redefine the architecture. Either preserve the invariant or explicitly document why a change is required.

---

## Ideas (`~/docker/ideas/`)

Immature ideas for projects and extensions live in `~/docker/ideas/`
(a symlink to `~/linuxadm/ideas/`; rules are in chapter 12 of that
repository's `AGENTS.md`). The idea files are written in Czech.

1. **Search before work**: `rg <topic> ~/docker/ideas/`. If an idea fits the
   task or its "Na co čeká" (waiting for) section is newly satisfied, tell the
   user. Do not implement it on your own.
1. **Out-of-scope ideas**: propose recording them in `~/docker/ideas/` using
   the template in `~/docker/ideas/INDEX.md`.

---

## 1. Mission

MORDECAI is a permissionless distributed AI network intended to operate **one shared intelligent system** across heterogeneous resources contributed by independent participants.

The distributed compute system is infrastructure.

The product is the AI.

Keep this distinction intact in code, APIs, naming, documentation, and architectural decisions.

MORDECAI is **not** primarily:

- a GPU rental marketplace;
- a generic decentralized cloud;
- a remote VM service;
- a cryptocurrency mining system;
- a Kubernetes replacement;
- a collection of unrelated hosted models.

When uncertain, optimize for the ability of MORDECAI to operate a shared, distributed, fault-tolerant intelligence.

---

# 2. Core Invariants

Agents MUST preserve these unless explicitly instructed otherwise.

## 2.1 Intelligence abstraction

User-facing APIs expose AI capabilities and execution budgets.

They MUST NOT require users to select physical infrastructure such as:

```text
GPU model
CPU count
RAM node
storage node
specific worker
```

Infrastructure selection belongs to the scheduler.

---

## 2.2 Heterogeneous nodes

Never assume all workers have:

- CUDA;
- NVIDIA hardware;
- x86-64;
- GPUs;
- equal RAM;
- equal bandwidth;
- low latency;
- permanent availability.

Capabilities must be discovered and represented explicitly.

---

## 2.3 Failure is expected

Remote nodes may disappear at any time.

Network partitions, timeouts, process crashes, suspend/resume, and stale capability data are normal operating conditions.

Distributed code must not treat worker loss as an impossible state.

---

## 2.4 No blockchain in the inference path

Do not put blockchain operations into:

- tensor transport;
- prompt processing;
- model execution;
- task scheduling;
- object retrieval;
- streaming responses.

MORDECAI v0 uses internal, non-transferable Compute Credits and requires no blockchain. Any future blockchain integration is a separate settlement/economic research decision and must remain outside the inference path.

---

## 2.5 Useful work and Compute Credits

Never credit a node merely because it claims resources or reports elapsed compute time.

MORDECAI v0 uses internal, non-transferable Compute Credits. They are not cryptocurrency, blockchain tokens, or user-to-user transferable assets.

Accounting should ultimately be based on useful contribution with an explicit verification/confidence method.

General trustless verification of heterogeneous neural-network computation is an open research problem. Agents MUST NOT silently treat `verification` as a solved universal primitive. Controlled prototypes may use trusted nodes; permissionless accounting requires workload-specific verification.

---

## 2.6 Privacy by protocol

Do not assume workers are trusted.

Do not expose more information to workers than required for their operation.

Avoid designs that unnecessarily reveal:

- user identity;
- complete prompts;
- complete responses;
- global task purpose;
- full model topology;
- unrelated worker identities.

---

## 2.7 Skills and knowledge are not authority

`skills/` describes reusable capabilities. `knowledge/` describes retrievable knowledge and its metadata.

Neither directory is a privilege boundary bypass.

Untrusted skill or knowledge content must not automatically:

- execute arbitrary code;
- access host files;
- read secrets;
- gain unrestricted network access;
- override system or runtime policy;
- bypass sandboxing.

Executable actions still require explicit runtime capabilities and policy checks.

---

## 2.8 Controlled learning

Production inference must not directly mutate production model weights.

Training and model promotion require a separate lifecycle:

```text
experience
    ↓
selection
    ↓
training
    ↓
candidate
    ↓
evaluation
    ↓
promotion / rejection
```

Rollback must remain possible.

---

# 3. Repository Structure

The intended high-level layout is:

```text
mordecai/
├── gateway/
├── core/
├── orchestrator/
├── scheduler/
├── node/
├── runtime/
├── network/
├── storage/
├── accounting/
├── privacy/
├── learning/
├── models/
├── skills/
├── knowledge/
├── api/
├── dashboard/
├── protocol/
├── docs/
├── tests/
├── README.md
├── ARCHITECTURE.md
└── AGENTS.md
```

Do not create new top-level directories casually.

Prefer placing functionality inside an existing subsystem.

If a new subsystem is architecturally justified, document its purpose.

---

# 4. Component Responsibilities

## `gateway/`

User and application ingress.

Responsibilities may include:

- API endpoints;
- authentication;
- sessions;
- request validation;
- budget handling;
- privacy-tier selection;
- streaming responses;
- conversion into internal task requests.

The gateway does not choose individual hardware workers directly.

---

## `core/`

Interfaces and runtime components associated with MORDECAI Core.

The Core is responsible conceptually for:

- high-level reasoning;
- planning;
- decomposition;
- synthesis.

Avoid coupling Core logic directly to physical node addresses.

---

## `orchestrator/`

Transforms AI objectives into executable task graphs.

Responsibilities:

- DAG creation;
- dependencies;
- specialist selection;
- parallel solver spawning;
- verifier insertion;
- retries at the logical task level;
- result aggregation.

The orchestrator decides **what work exists**.

The scheduler decides **where it runs**.

Do not merge these concepts accidentally.

---

## `scheduler/`

Maps executable work to available resources.

Responsibilities:

- capability matching;
- locality;
- resource availability;
- model shard locality;
- latency;
- bandwidth;
- reliability;
- privacy constraints;
- cost;
- replication;
- failover candidates.

The scheduler must be topology-aware.

---

## `node/`

Contributor daemon.

Responsibilities:

- node identity;
- capability discovery;
- resource policy;
- benchmarks;
- job lifecycle;
- local cache;
- telemetry;
- receipts;
- sandbox interaction.

Node code must be conservative about host resources.

Never assume the contributor wants MORDECAI to consume everything available.

---

## `runtime/`

Execution abstraction above hardware-specific backends.

Possible backends:

- CPU;
- CUDA;
- ROCm;
- Metal;
- ONNX Runtime;
- mobile NPU runtimes;
- WebAssembly;
- future accelerator APIs.

Hardware-specific behavior should remain behind interfaces where practical.

---

## `network/`

Peer communication and transport.

Responsibilities may include:

- QUIC transport;
- connection lifecycle;
- NAT traversal;
- relays;
- peer discovery;
- encrypted channels;
- multiplexing;
- topology/privacy mechanisms.

Do not invent custom cryptography.

---

## `storage/`

Content-addressed distributed object storage.

Objects may include:

- model shards;
- checkpoints;
- experts;
- datasets;
- embeddings;
- task artifacts;
- experience records.

Prefer immutable content-addressed objects.

Mutable namespaces should resolve to immutable versions.

---

## `accounting/`

Useful-work measurement and Compute Credit accounting.

Responsibilities:

- work receipts;
- verification results;
- contribution accounting;
- epoch aggregation;
- credit accounting data.

Do not couple Compute Credit accounting to a blockchain or transferable token.

---

## `privacy/`

Privacy-preserving protocols and experimental implementations.

Examples:

- relay routing;
- topology hiding;
- secret sharing;
- MPC;
- anonymous credentials;
- zero-knowledge mechanisms.

Experimental cryptography must be clearly marked as experimental.

Never describe an experimental privacy mechanism as secure without appropriate review.

---

## `learning/`

Continual-learning pipeline.

Responsibilities:

- experience capture;
- filtering;
- datasets;
- candidate training;
- evaluation;
- promotion metadata;
- rollback metadata.

User data must not silently become training data.

---

## `models/`

Model metadata and model integration.

Do not commit enormous model weights directly into Git.

Use manifests and content-addressed references.

---

## `skills/`

Definitions of reusable capabilities MORDECAI can invoke.

Skills may describe:

- tool workflows;
- specialist procedures;
- capability metadata;
- input/output schemas;
- execution requirements;
- policy and permission requirements;
- reusable orchestration patterns.

A skill describes **what MORDECAI can do**, not unrestricted authority to execute arbitrary code.

Skills must use explicit interfaces and declared permissions. Content placed in `skills/` MUST NOT automatically gain access to contributor hosts, secrets, network resources, or privileged runtime capabilities.

Executable skill implementations must still pass through the normal runtime, sandbox, scheduler, policy, and security boundaries.

---

## `knowledge/`

Definitions and small repository-managed assets describing **what MORDECAI can know or retrieve**.

Appropriate contents include:

- knowledge-source definitions;
- retrieval configuration;
- schemas;
- ingestion rules;
- provenance metadata;
- small curated static knowledge assets;
- indexes or manifests that reference external/content-addressed data.

Do not use Git as the primary storage layer for:

- large datasets;
- embedding databases;
- vector-store runtime state;
- model-generated caches;
- private user data;
- secrets;
- large mutable knowledge corpora.

Large knowledge objects belong in the content-addressed/distributed storage layer and should be referenced by immutable identifiers where practical.

Knowledge is data, not authority. A document, retrieved chunk, or knowledge source MUST NOT automatically gain permission to execute code, invoke privileged tools, or override system policy.

---

## `api/`

Shared API schemas and protocol-facing interfaces.

Public interfaces should be versioned.

---

## `dashboard/`

Human-readable aggregate network status.

Never expose privacy-sensitive worker topology or user activity.

---

## `protocol/`

Wire schemas and protocol definitions shared between implementations.

Protocol evolution must preserve explicit versioning.

---

# 5. Development Priorities

Unless a task explicitly says otherwise, prioritize engineering in this order:

1. correctness;
2. testability;
3. fault tolerance;
4. observability;
5. security;
6. interoperability;
7. performance;
8. elegance.

Do not sacrifice correctness for speculative optimization.

Do not implement advanced decentralization before a simpler implementation can prove the underlying execution model.

---

# 6. Current Implementation Strategy

The project should evolve incrementally.

## Stage A — Controlled prototype

Assume a small number of trusted development nodes.

Prove:

- node registration;
- capability discovery;
- task dispatch;
- distributed object transfer;
- execution;
- failure detection;
- accounting records.

Security shortcuts used during this stage MUST be documented.

---

## Stage B — Distributed inference

Prove that one model can execute across multiple machines.

Important metrics:

- tokens/second;
- time-to-first-token;
- stage latency;
- inter-node bandwidth;
- transferred bytes/token;
- worker failure recovery time;
- cache hit ratio.

Do not optimize based only on local benchmarks.

WAN behavior matters.

---

## Stage C — Permissionless execution

Introduce:

- untrusted workers;
- verification;
- reputation;
- stronger sandboxing;
- adversarial testing.

---

## Stage D — Advanced privacy and economics

Only after the execution architecture is measurable and stable should the project aggressively optimize:

- MPC;
- anonymous accounting research;
- future transferable-asset economics;
- privacy overlays;
- decentralized governance.

Do not let blockchain or token engineering become a substitute for distributed AI engineering.

---

# 7. Coding Standards

## 7.1 General

Code should be:

- explicit;
- boring where possible;
- testable;
- observable;
- deterministic where practical;
- resistant to partial failure.

Avoid clever abstractions with no demonstrated need.

Distributed systems already provide enough creative suffering.

---

## 7.2 Languages

Preferred direction:

### Rust

Suitable for:

- node daemon;
- networking;
- scheduler components;
- protocol implementation;
- performance-sensitive runtime components.

### Python

Suitable for:

- model execution;
- ML integration;
- experiments;
- training;
- evaluation;
- research tooling.

### Go

Acceptable for control-plane services where it provides a practical advantage.

Do not rewrite working components merely to satisfy language preference.

---

## 7.3 Python

Target modern supported Python.

Prefer:

- type hints;
- dataclasses or validated schema models;
- explicit async boundaries;
- structured logging;
- pytest;
- small modules.

Avoid hidden global mutable state.

Public functions should have meaningful types.

Example:

```python
async def schedule_task(
    task: Task,
    candidates: list[WorkerCapability],
) -> Placement:
    ...
```

Prefer this over dictionaries whose schema exists only in someone's memory.

---

## 7.4 Rust

Prefer:

- stable Rust;
- `rustfmt`;
- `clippy`;
- explicit error types;
- structured tracing;
- bounded concurrency;
- cancellation-aware async operations.

Avoid `unwrap()` in network-facing production paths unless the invariant is genuinely impossible to violate and documented.

---

# 8. Data Schemas

Protocol and persisted data structures must be versionable.

Good:

```json
{
  "schema_version": 1,
  "task_id": "...",
  "operation": "...",
  "requirements": {}
}
```

Bad:

```json
{
  "gpu": "yes",
  "stuff": {}
}
```

Identifiers should be opaque unless semantics are intentionally part of the protocol.

---

# 9. Resource Representation

Do not model the network solely as GPU inventory.

Prefer abstract capabilities:

```text
scalar_compute
tensor_compute
vector_compute
fast_memory
bulk_memory
persistent_storage
network_bandwidth
cryptographic_compute
```

Vendor-specific properties belong inside capability metadata.

Example:

```yaml
tensor_compute:
  backend: cuda
  architecture: sm_90
  memory_bytes: 85899345920
  precision:
    - fp32
    - fp16
    - bf16
    - int8
```

The scheduler should reason about requirements, not marketing names.

---

# 10. Units

Always use explicit units.

Preferred internal conventions:

- bytes, not ambiguous MB;
- nanoseconds or milliseconds with type-safe representation;
- bits/s or bytes/s explicitly;
- watts explicitly;
- FLOP/s with precision specified where meaningful.

Do not mix:

```text
GB
GiB
MB
MiB
Gbps
GB/s
```

without explicitly representing the unit.

Human-facing interfaces may format values conveniently.

---

# 11. Time

Distributed systems and wall clocks are enemies pretending to be friends.

Use:

- monotonic clocks for durations and timeouts;
- UTC timestamps for persisted events;
- explicit expiration;
- bounded clock-skew assumptions only when unavoidable.

Never use wall-clock subtraction for critical timeout logic.

---

# 12. Networking

All network operations must have:

- timeout behavior;
- cancellation;
- bounded retries;
- backoff;
- maximum message sizes;
- validation;
- structured error reporting.

Never implement infinite retry loops.

Retries should distinguish between:

- transient failure;
- permanent rejection;
- invalid request;
- incompatible protocol;
- unavailable resource.

---

# 13. QUIC

QUIC is a preferred candidate for peer transport because it provides:

- encrypted transport;
- multiplexed streams;
- connection migration;
- modern congestion control;
- reduced head-of-line blocking between streams.

Do not assume one QUIC stream should carry every traffic class.

Separate logical streams for:

- control;
- tensors;
- object transfer;
- receipts;
- telemetry

where appropriate.

Protocol design must remain benchmark-driven.

---

# 14. Storage Rules

Content-addressed objects should be immutable.

A stored object must be verified against its digest before being trusted.

Never execute a downloaded runtime artifact solely because its filename looks correct.

Metadata should identify:

- hash algorithm;
- digest;
- size;
- object type;
- encoding;
- optional chunking information.

---

# 15. Model Artifacts

Model weights must not be committed directly into the source repository unless they are tiny test fixtures.

Use manifests.

Example:

```yaml
model:
  id: mordecai-core-test
  version: 1

shards:
  - digest: blake3:...
    size: 2147483648
    stage: 0

  - digest: blake3:...
    size: 2147483648
    stage: 1
```

Tests should use synthetic or small openly licensed models.

---

# 16. Security

Assume all externally supplied input is malicious until validated.

This includes:

- API requests;
- peer messages;
- model manifests;
- object metadata;
- capability advertisements;
- receipts;
- task results;
- dashboard input;
- training artifacts.

Use strict parsing.

Enforce limits before allocation when possible.

---

# 17. Cryptography

Never invent cryptographic primitives.

Use established libraries and protocols.

Do not:

- create custom encryption;
- design a new hash function;
- implement ad-hoc signatures;
- reuse nonces incorrectly;
- store private keys in source code;
- log secrets.

Experimental cryptographic protocols require:

- threat model;
- references;
- test vectors where available;
- explicit experimental status.

---

# 18. Secrets

Secrets must never be committed.

Examples:

- private keys;
- API tokens;
- wallet seeds;
- database passwords;
- TLS private keys;
- production credentials.

Provide `.example` configuration files using dummy values.

---

# 19. Logging

Logs are part of the privacy surface.

Do not log:

- complete prompts;
- complete responses;
- authorization tokens;
- private keys;
- wallet seeds;
- raw secret shares;
- unnecessary user identifiers.

Prefer structured logs.

Example fields:

```text
task_id
node_id
stage
duration_ms
bytes_sent
bytes_received
result
error_code
```

Task identifiers should be opaque.

---

# 20. Metrics

Every distributed component should expose useful metrics.

Important categories:

### Node

- active jobs;
- resource utilization;
- cache usage;
- bytes transferred;
- execution latency;
- failures.

### Scheduler

- queue depth;
- placement latency;
- rejected placements;
- retry count;
- locality score;
- estimated vs actual execution time.

### Network

- RTT;
- throughput;
- packet loss where available;
- reconnects;
- stream failures.

### Model execution

- time to first token;
- tokens/second;
- stage latency;
- transferred bytes/token;
- cache hit rate.

Avoid labels with unbounded cardinality.

---

# 21. Testing

Every significant change should include tests.

## Unit tests

Use for:

- parsers;
- scheduling rules;
- accounting calculations;
- protocol encoding;
- object hashing;
- capability matching.

## Integration tests

Use for:

- gateway → orchestrator;
- scheduler → node;
- node → runtime;
- storage replication;
- model stage transfer.

## Failure tests

Actively test:

- worker termination;
- packet delay;
- packet loss;
- corrupted objects;
- stale capabilities;
- duplicate messages;
- task timeout;
- partial result delivery.

A distributed system tested only on localhost is a distributed system with excellent self-esteem.

---

# 22. Simulation

Where practical, build simulation hooks for:

- node churn;
- latency distributions;
- bandwidth limits;
- heterogeneous accelerators;
- malicious nodes;
- storage loss;
- scheduler policies.

Scheduler changes should be testable without requiring a physical global network.

---

# 23. Benchmarks

Performance claims require reproducible measurements.

A benchmark should record:

- commit;
- hardware;
- model;
- precision;
- topology;
- worker count;
- RTT;
- bandwidth;
- batch size;
- prompt length;
- generated tokens;
- relevant runtime settings.

Never publish "2× faster" without defining faster than what.

---

# 24. Scheduler Changes

Scheduler modifications are architecture-sensitive.

Before changing placement logic, consider:

- compute;
- memory;
- model locality;
- data locality;
- bandwidth;
- latency;
- reliability;
- privacy;
- energy;
- execution budget.

A scheduler optimization that improves FLOP utilization while doubling WAN transfer may be a regression.

---

# 25. Privacy Changes

For privacy-sensitive code, document:

```text
protected information:
adversary:
trust assumptions:
metadata leaked:
collusion threshold:
failure behavior:
```

Do not use the word "anonymous" when the mechanism only provides pseudonymity.

Do not use the word "private" without stating from whom.

---

# 26. Accounting Changes

Accounting must distinguish:

- claimed capacity;
- reserved capacity;
- executed work;
- verified work;
- settled work.

These are not interchangeable.

Example lifecycle:

```text
advertised
    ↓
scheduled
    ↓
executed
    ↓
verified
    ↓
credited
    ↓
settled
```

Only later states should produce economic finality.

---

# 27. Learning Changes

Training-related contributions require particular care.

Do not automatically persist arbitrary production prompts into training datasets.

Training artifacts should record provenance and policy.

Candidate models must not overwrite production artifacts.

Use immutable versions.

Example:

```text
core/v48
core/v49-candidate-a
core/v49-candidate-b
core/v49-candidate-c
```

Promotion updates a reference to an immutable artifact.

---

# 28. API Design

Public APIs should be boring and familiar.

Where practical, provide compatibility with common AI API conventions.

Example:

```http
POST /v1/chat/completions
POST /v1/responses
POST /v1/embeddings
```

MORDECAI-specific extensions should use explicit fields.

Example:

```json
{
  "mode": "deep",
  "max_budget": 100,
  "privacy": "private"
}
```

Do not expose scheduler internals in the public API.

---

# 29. Error Handling

Errors should be machine-readable.

Prefer:

```json
{
  "error": {
    "code": "NO_COMPATIBLE_WORKER",
    "message": "No worker currently satisfies the task requirements.",
    "retryable": true
  }
}
```

Avoid exposing internal addresses, stack traces, peer identities, or secrets to public clients.

---

# 30. Configuration

Configuration should be explicit and validated at startup.

Suggested precedence:

```text
defaults
  ↓
configuration file
  ↓
environment
  ↓
CLI arguments
```

Security-sensitive defaults should be conservative.

Do not silently enable public listening interfaces.

---

# 31. Backward Compatibility

Protocol-breaking changes require version changes.

Do not assume every node upgrades simultaneously.

Permissionless networks inherently contain mixed versions.

Where practical:

- negotiate capabilities;
- negotiate protocol versions;
- reject unsupported versions clearly;
- allow rolling upgrades.

---

# 32. Dependencies

Add dependencies deliberately.

Before adding one, consider:

- maintenance status;
- security history;
- license;
- transitive dependency size;
- platform support;
- whether the standard library already solves the problem.

Do not add a framework to avoid writing twenty lines of understandable code.

---

# 33. Licensing

Do not introduce dependencies, datasets, model weights, or copied code with unclear licensing.

Record licenses for:

- source dependencies;
- models;
- datasets;
- runtime binaries;
- generated assets.

The final MORDECAI licensing strategy is not yet fixed, so avoid decisions that unnecessarily constrain it.

---

# 34. Documentation

Architecture-relevant code must be documented.

Update documentation when changing:

- protocol messages;
- node capabilities;
- scheduler behavior;
- storage format;
- model manifests;
- privacy assumptions;
- accounting semantics.

Do not let implementation and architecture silently diverge.

---

# 35. Commit Discipline

Prefer focused changes.

A good commit should answer one question.

Examples:

```text
feat(node): add capability discovery
feat(storage): verify BLAKE3 object digests
fix(scheduler): retry placement after stale worker removal
test(network): simulate interrupted tensor stream
docs(protocol): document worker handshake v1
```

Avoid mixing:

- formatting;
- unrelated refactors;
- dependency upgrades;
- new features

in one change unless necessary.

---

# 36. Agent Workflow

Before writing code:

1. read the relevant documentation;
2. inspect existing implementation;
3. identify subsystem boundaries;
4. understand existing tests;
5. state assumptions internally;
6. make the smallest coherent change.

After writing code:

1. format;
2. lint;
3. run relevant unit tests;
4. run relevant integration tests;
5. inspect failures rather than hiding them;
6. update documentation if behavior changed;
7. summarize what changed.

---

# 37. Do Not Fake Success

Agents must never:

- claim tests passed without running them;
- invent benchmark numbers;
- invent available hardware;
- hide failing tests;
- replace a real integration with a mock and claim completion;
- disable validation merely to make tests green;
- silently swallow distributed-system errors.

If a test cannot run because hardware or infrastructure is unavailable, say so explicitly.

---

# 38. Avoid Premature Complexity

MORDECAI has an ambitious long-term architecture.

That does not mean version 0.1 needs:

- a custom blockchain;
- full MPC inference;
- a global DHT;
- mobile NPU support;
- Byzantine consensus;
- five scheduler implementations;
- autonomous model evolution.

Build the smallest system that proves the next architectural claim.

For example:

```text
v0.1:
two nodes can execute one distributed model reliably

before:

v0.1:
we invented civilization again in Rust
```

---

# 39. Preferred PoC

When no more specific implementation direction exists, optimize toward this milestone:

> Run a single model across multiple independently connected MORDECAI nodes and expose it through an OpenAI-compatible API.

The PoC should demonstrate:

- node capability registration;
- model shard placement;
- content-addressed shard retrieval;
- pipeline execution;
- activation transfer;
- streaming output;
- basic scheduler decisions;
- timeout and worker failure handling;
- metrics;
- execution receipts.

This validates more of MORDECAI's thesis than implementing a token first.

---

# 40. Definition of Done

A feature is not done merely because the happy path works.

For distributed components, Definition of Done normally includes:

- implementation;
- input validation;
- error handling;
- timeout behavior;
- cancellation where relevant;
- unit tests;
- integration tests where relevant;
- logging;
- metrics where relevant;
- documentation;
- no known secret leakage.

---

# 41. Architectural Questions

When choosing between designs, ask:

1. Does this help operate one shared AI?
2. Does it work with heterogeneous hardware?
3. What happens when a worker disappears?
4. What crosses the network?
5. How many bytes cross the WAN per unit of useful work?
6. What does the worker learn about the task?
7. How is the result verified?
8. How is useful contribution measured?
9. Can this be upgraded without every node upgrading simultaneously?
10. Can we test it without owning a datacenter?

If a proposal cannot answer these questions, it is probably not ready.

---

# 42. Naming

Use the project name:

**MORDECAI**

The typographic interpretation:

**MORDEC·AI**

may be used in branding, but identifiers should normally use `mordecai`.

Examples:

```text
mordecai-node
mordecai-gateway
mordecai-scheduler
mordecai-runtime
MORDECAI_PROTOCOL_VERSION
```

Avoid gratuitous variations of the project name.

---

# 43. Final Rule

MORDECAI is ambitious enough without accidental complexity.

Build measurable systems.

Assume networks fail.

Assume nodes lie.

Verify objects.

Verify work.

Keep secrets out of logs.

Keep blockchain out of tensor paths.

Keep hardware details below the intelligence API.

And when a design requires the entire Internet to behave like localhost, the design is wrong.

## Validation according to change impact

- Select checks from the actual impact and build inputs, not file extensions.
- Changes limited to project documentation that is neither used at runtime
  nor included in published output require content, formatting, link, and
  diff checks. Do not build the application or images, regenerate assets,
  or run application integration and end-to-end tests solely for such edits.
- Code, dependency, configuration, template, published-content, and runtime
  changes require the relevant project tests and builds. Markdown used as
  website content or production prompts and instructions is not automatically
  documentation-only.
- Reuse a tested artifact only when the existing release workflow verifies
  identical build inputs and traceable artifact provenance. This rule does
  not introduce a new release system.
