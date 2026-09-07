# MORDECAI Roadmap

**Status:** Draft  
**Planning style:** Capability-driven, not date-driven

MORDECAI should advance by proving architectural claims in increasing order of difficulty.

The roadmap deliberately puts distributed AI execution before transferable-token engineering, governance theater, or advanced cryptography.

---

## North Star

Build one shared intelligent system whose useful compute, memory, storage, and network capacity can be supplied by independent participants.

The decisive early milestone is:

> **Multiple machines connected over real networks reliably execute one model that is presented to the user as one AI.**

---

# Phase 0 — Repository and local skeleton

## Goal

Create an implementation foundation that can evolve without immediately becoming a distributed monolith.

## Deliverables

- repository layout;
- build/test tooling;
- gateway skeleton;
- node daemon skeleton;
- scheduler skeleton;
- protocol schemas;
- local object store;
- metrics;
- structured logging;
- CI.

## Exit criteria

- two local node processes register capabilities;
- scheduler sees both;
- a test task can be dispatched;
- task result is returned;
- failure is visible in metrics/logs;
- CI executes tests.

---

# Phase 1 — Distributed execution v0

## Goal

Execute useful work across machines over an actual network.

## Deliverables

- authenticated node connections;
- QUIC or selected transport;
- capability advertisement;
- leases/heartbeats;
- task offer/accept/reject;
- timeouts/cancellation;
- content-addressed object transfer;
- execution receipts.

## Tests

- artificial latency;
- bandwidth limiting;
- worker termination;
- duplicate messages;
- corrupt objects;
- stale node state.

## Exit criteria

A task can be reliably scheduled between independent hosts and survive expected failure modes.

---

# Phase 2 — Sharded model PoC

## Goal

Prove MORDECAI's central technical thesis.

Run one model across multiple nodes such that no selected worker needs to hold the complete model in accelerator memory.

## Initial target

Illustrative development target:

```text
4–8 workers
heterogeneous where practical
Internet/WAN-like links
pipeline-partitioned model
OpenAI-compatible API
```

The exact model size depends on available development hardware.

## Deliverables

- model manifest;
- model shard objects;
- stage placement;
- shard caching;
- activation streams;
- pipeline inference;
- microbatching;
- streamed generation;
- stage replicas/failover.

## Measure

- time to first token;
- tokens/s;
- per-stage latency;
- bytes transferred/token;
- shard download time;
- cache hit ratio;
- failure recovery time.

## Exit criteria

A user sends one request to MORDECAI and receives a model response produced by a model physically partitioned across multiple nodes.

---

# Phase 3 — WAN-aware scheduler

## Goal

Stop treating every node as if it lived in the same rack.

## Deliverables

Scheduler considers:

- RTT;
- bandwidth;
- accelerator capability;
- memory;
- model locality;
- reliability;
- availability;
- expected execution cost.

Introduce logical swarms/regions.

## Experiments

Compare:

- random placement;
- compute-only placement;
- locality-aware placement;
- latency/bandwidth-aware placement.

## Exit criteria

Scheduler policy measurably improves end-to-end inference performance or reliability over naive placement.

---

# Phase 4 — Hierarchical intelligence

## Goal

Make MORDECAI an orchestrated intelligence rather than only a distributed forward pass.

## Deliverables

- task DAG;
- Core planner interface;
- specialist registry;
- skill registry and skill metadata;
- knowledge/retrieval layer;
- knowledge-source manifests and provenance;
- deterministic tool interface;
- parallel solvers;
- critics/verifiers;
- synthesis.

Initial specialists:

- math;
- code;
- retrieval/research.

Initial skill/knowledge target:

- load repository-managed skill definitions;
- resolve knowledge-source definitions;
- retrieve a bounded knowledge context;
- expose explicit skill permissions;
- keep executable actions behind runtime/sandbox policy.

## Exit criteria

A complex task is decomposed into multiple independent operations, executed in parallel where useful, verified, and synthesized into one answer.

---

# Phase 5 — Permissionless alpha

## Goal

Allow unknown contributors to join without trusting their claims or results.

## Deliverables

- public node onboarding;
- node identity;
- automatic benchmarking;
- capability verification;
- work verification;
- challenge jobs;
- reputation;
- stronger sandbox;
- signed receipts;
- abuse controls.

## Exit criteria

An untrusted node can join and perform useful work without the scheduler blindly trusting its self-reported capability or result.

---

# Phase 6 — Heterogeneous resource fabric

## Goal

Use more than GPUs.

## CPU

Add workloads such as:

- preprocessing;
- tokenization;
- retrieval;
- symbolic verification;
- compilation/testing;
- orchestration helpers.

## RAM

Add:

- model shard cache;
- KV/cache experiments;
- regional memory service.

## Storage

Add:

- distributed immutable objects;
- replication;
- erasure coding experiments;
- repair;
- possession verification.

## Network

Add:

- relay nodes;
- model distribution caches;
- topology measurements.

## NPU/mobile

Add:

- capability abstraction;
- small models/experts;
- embeddings;
- reranking;
- speculative decoding;
- device policy.

## Exit criteria

The scheduler can allocate different task types to materially different resource classes.

---

# Phase 7 — Compute Credit accounting

## Goal

Translate measured and sufficiently verified AI work into internal MORDECAI Compute Credits.

Compute Credits in this phase are:

- internal;
- non-transferable;
- not cryptocurrency;
- not blockchain tokens;
- not exchangeable by the protocol;
- stored in conventional accounting infrastructure.

Do not launch a transferable token merely because the database has acquired a `balance` column.

## Deliverables

- resource/work measurement;
- work receipts;
- verification/confidence metadata;
- contributor credit;
- user debit;
- immutable accounting history;
- replay/double-credit prevention;
- economic telemetry.

## Research requirement

General trustless verification of heterogeneous neural-network computation remains an open research problem.

Phase 7 must document which workload classes are:

- trusted;
- deterministic;
- redundantly checked;
- challenge-verified;
- statistically checked;
- not yet safely creditable in a permissionless environment.

## Exit criteria

The network can explain why each participant was credited or debited from auditable work records, and the limits of the verification method are explicit.


---

# Phase 8 — Economic and regulatory research

## Goal

Determine whether MORDECAI needs any transferable economic asset at all.

The working system continues to use internal Compute Credits.

## Research

Evaluate:

- whether transferability is technically necessary;
- contributor/user economic behavior observed from Compute Credits;
- useful-work verification maturity;
- fraud and Sybil resistance;
- settlement requirements;
- privacy requirements;
- legal and regulatory classification;
- operational and governance consequences.

Possible future mechanisms may include conventional payments, external settlement systems, or a blockchain-based asset, but none is assumed.

## Exit criteria

No implementation milestone requires a token.

A transferable asset proceeds only after there is a demonstrated technical need, credible verification model, and appropriate legal/regulatory review.


---

# Phase 9 — Continual learning

## Goal

Allow MORDECAI to improve from experience safely.

## Deliverables

- experience schema;
- consent/policy controls;
- provenance;
- experience lake;
- dataset selection;
- distributed training jobs;
- candidate registry;
- evaluation harness;
- promotion;
- rollback.

## Model cadence

Potential pattern:

```text
semantic memory      continuous
episodic memory      continuous
adapters             hours/days
specialists          days
core                 longer controlled cycles
```

## Exit criteria

A candidate specialist or Core version can be trained from approved experience, evaluated against the current version, promoted, and rolled back reproducibly.

---

# Phase 10 — Privacy overlay

## Goal

Reduce information available to individual workers and network observers.

## Deliverables

- relay topology;
- hidden direct relationships;
- opaque task identities;
- padded message classes;
- batching;
- randomized placement;
- metadata measurement.

## Exit criteria

A worker can execute selected jobs while learning materially less about user identity, global task context, and neighboring topology than in the standard execution mode.

---

# Phase 11 — Dark Compute research

## Goal

Explore cryptographic execution in which individual contributors do not receive useful plaintext model/task state.

## Research tracks

- secret-shared activations;
- secret-shared weights;
- transformer MPC kernels;
- Beaver-triple/preprocessing systems;
- private expert routing;
- secret-shared KV cache;
- anonymous credentials;
- unlinkable work accounting;
- hybrid FHE/MPC.

## Required measurements

For every mechanism:

- latency multiplier;
- bandwidth multiplier;
- memory multiplier;
- preprocessing cost;
- collusion threshold;
- failure recovery;
- information leakage.

## Exit criteria

No fixed production exit criterion yet.

This phase is research and must be benchmark-driven.

---

# Phase 12 — Control-plane decentralization

## Goal

Reduce reliance on privileged central scheduling and registry infrastructure without sacrificing performance.

Possible directions:

- regional schedulers;
- federated scheduling;
- verifiable placement;
- distributed registries;
- privacy-preserving rendezvous;
- governance separation.

Decentralization is not measured by the number of microservices.

## Exit criteria

A defined control-plane failure or operator compromise no longer represents a global single point of control for the targeted function.

---

# Cross-cutting tracks

Some work runs across all phases.

## Security

- fuzzing;
- dependency review;
- release signing;
- sandbox hardening;
- threat-model updates;
- incident response.

## Observability

- metrics;
- tracing;
- performance regression tracking;
- network telemetry;
- public aggregate dashboard.

## Developer experience

- reproducible dev environment;
- test clusters;
- network simulation;
- protocol inspection tools;
- documentation.

## Compatibility

- OpenAI-compatible API where practical;
- multiple accelerator backends;
- protocol version negotiation.

---

# Things deliberately postponed

Until distributed inference is proven, avoid spending major engineering effort on:

- custom blockchain or transferable token;
- elaborate DAO governance;
- exchange integrations;
- speculative token price mechanics;
- full custom distributed filesystem semantics;
- global Byzantine consensus;
- production MPC transformer inference;
- dozens of model modalities.

The project already contains enough hard problems. We do not need to collect them like Pokémon.

---

# First concrete milestone

The immediate engineering milestone is:

```text
Client
  │
  ▼
Gateway
  │
  ▼
Scheduler
  │
  ├────────► Node A: model stage 0
  │                    │
  │                    ▼
  ├────────► Node B: model stage 1
  │                    │
  │                    ▼
  └────────► Node C: model stage 2
                       │
                       ▼
                    output
```

with:

- content-addressed model shards;
- real network transport;
- metrics;
- timeout handling;
- worker failover;
- streamed response.

When this works outside localhost, MORDECAI has begun proving its thesis.
