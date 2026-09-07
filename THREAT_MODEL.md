# MORDECAI Threat Model

**Status:** Draft  
**Scope:** Architecture and early protocol

MORDECAI is designed for a future in which independent participants contribute compute, memory, storage, and network resources.

Therefore:

> A worker is not trusted merely because it successfully connected.

This document identifies assets, adversaries, trust boundaries, threats, and intended mitigations.

It does not claim that all mitigations already exist.

---

## 1. Security goals

MORDECAI aims to protect:

- user requests and results;
- model artifacts;
- network integrity;
- useful-work accounting;
- contributor rewards;
- node host systems;
- training integrity;
- protocol availability;
- privacy-sensitive topology;
- software supply chain.

Long-term privacy goal:

> No individual contributor should have enough information to reconstruct the complete prompt, response, or model.

This is a design objective, not a claim about the initial PoC.

---

## 2. Assets

### User assets

- prompts;
- uploaded data;
- outputs;
- account/session credentials;
- balances;
- private metadata.

### Network assets

- node identities;
- reputation;
- scheduler state;
- routing/topology information;
- accounting records;
- Compute Credit accounting records.

### Model assets

- model shards;
- complete model topology;
- checkpoints;
- expert routing;
- KV cache;
- training artifacts;
- evaluation results.

### Contributor assets

- host operating system;
- files;
- private network;
- accelerator memory;
- credentials;
- bandwidth;
- earned balance.

---

## 3. Adversaries

Assume the existence of:

- malicious users;
- malicious workers;
- malicious node operators;
- Sybil operators;
- colluding workers;
- compromised workers;
- malicious relays;
- malicious storage nodes;
- malicious validators;
- compromised control-plane services;
- supply-chain attackers;
- network observers.

Some adversaries may control many identities.

---

## 4. Trust boundaries

Major boundaries:

```text
User
 │
 ▼
Gateway
 │
 ▼
Control Plane
 │
 ▼
Permissionless Network
 │
 ├── Worker
 ├── Storage Node
 ├── Relay
 └── Verifier
```

The permissionless network is untrusted.

Even inside the control plane, compromise must be considered.

---

## 5. Worker threats

A malicious worker may:

- return incorrect results;
- claim nonexistent hardware;
- lie about performance;
- claim work it did not perform;
- retain task data;
- inspect activations;
- attempt model reconstruction;
- replay old results;
- selectively fail tasks;
- manipulate timing;
- collude with neighboring stages.

Mitigations include:

- verification;
- challenge tasks;
- replication;
- reputation;
- signed receipts;
- minimal disclosure;
- model sharding;
- future secret sharing/MPC.

---

## 6. Sybil attacks

One operator may create thousands of apparent nodes.

Node count is therefore not equivalent to decentralization.

Mitigation candidates:

- useful-work history;
- economic stake where appropriate;
- resource verification;
- challenge work;
- rate limits;
- diversity-aware scheduling;
- independent network/identity signals where privacy permits.

Do not rely on IP address uniqueness as identity.

NAT exists. IPv6 exists. Botnets also exist. Life is complicated.

---

## 7. Incorrect computation

Threat:

A worker returns a plausible but wrong result and requests payment.

Mitigations depend on workload:

- deterministic recomputation;
- replicated execution;
- challenge inputs;
- statistical comparison;
- downstream consistency checks;
- specialized verifiers.

Verification cost should be lower than blindly duplicating all work where possible.

However, **general trustless verification of heterogeneous neural-network computation is an open research problem**. GPU/accelerator results may differ because of precision, kernels, drivers, hardware, and nondeterministic execution. Exact comparison is therefore not universally available, while tolerance-based comparison creates an adversarial acceptance boundary.

MORDECAI must not describe work verification as solved until a workload-specific mechanism has been implemented, benchmarked, and threat-modeled.

---

## 8. Model theft

A worker hosting a model shard may attempt to reconstruct more of the model.

Basic sharding limits immediate exposure but is not sufficient against:

- long-term shard collection;
- collusion;
- malicious scheduling;
- repeated assignment.

Mitigations may include:

- shard assignment policy;
- rotation;
- limiting co-location;
- encryption at rest;
- authenticated manifests;
- secret-shared weights;
- MPC;
- monitoring unusual shard acquisition.

A public open-weight model has different confidentiality requirements from a controlled Core model.

---

## 9. Prompt and output disclosure

A pipeline worker may infer information from:

- tokens;
- activations;
- tensor shapes;
- timing;
- neighboring stages.

Encrypted transport protects data in transit but does not protect plaintext from the endpoint performing ordinary computation.

Long-term mitigations:

- task opacity;
- topology hiding;
- secret sharing;
- MPC compute cells;
- padding;
- batching;
- traffic shaping.

Privacy claims must state the relevant adversary.

---

## 10. Metadata analysis

Even encrypted payloads leak metadata:

- timing;
- message size;
- communication graph;
- session duration;
- model-stage patterns.

Mitigation candidates:

- relays;
- fixed/padded frame classes;
- batching;
- randomized scheduling;
- multiplexing;
- cover traffic for high-privacy modes.

These impose real performance costs.

Privacy tiers should make that trade-off explicit.

---

## 11. Collusion

If privacy relies on multiple independent workers, collusion matters.

For a 3-party secret-sharing cell, document whether security assumes:

- 1 of 3 may be malicious;
- fewer than 2 collude;
- honest majority;
- malicious-majority tolerance.

Scheduler diversity is part of the security mechanism.

Do not place all privacy-cell participants under one obvious operator when avoidable.

---

## 12. Storage attacks

A storage node may:

- return corrupted data;
- omit data;
- serve stale versions;
- claim possession without storing data;
- selectively censor objects.

Mitigations:

- content-address verification;
- signed manifests;
- replication;
- erasure coding;
- possession challenges;
- repair;
- immutable versioning.

Never trust an object because the server said `200 OK`.

---

## 13. Manifest attacks

A malicious manifest may point to:

- attacker-controlled code;
- wrong model shards;
- incompatible tensor layouts;
- oversized objects;
- malicious runtime artifacts.

Mitigations:

- authenticated manifests;
- schema validation;
- size limits;
- allowlisted runtime types;
- sandboxing;
- digest verification.

---

## 14. Node host compromise

MORDECAI tasks must not gain arbitrary access to contributor hosts.

Risks:

- filesystem access;
- credential theft;
- network pivoting;
- device abuse;
- privilege escalation.

Mitigations:

- process isolation;
- containers;
- rootless execution;
- seccomp/capability reduction where available;
- filesystem namespaces;
- network policy;
- WebAssembly for suitable jobs;
- resource limits;
- no host secrets in worker environment.

Containerization alone is not a complete security boundary.

---

## 15. Resource exhaustion

Attackers may submit work designed to exhaust:

- RAM;
- VRAM;
- disk;
- file descriptors;
- streams;
- CPU;
- bandwidth;
- parser recursion;
- queue capacity.

Mitigations:

- quotas;
- declared sizes;
- bounded allocation;
- rate limiting;
- deadlines;
- admission control;
- per-peer limits;
- backpressure.

Validate before allocating whenever possible.

---

## 16. Network attacks

Threats include:

- replay;
- spoofing;
- downgrade;
- connection floods;
- stream exhaustion;
- malformed frames;
- amplification.

Mitigations:

- authenticated encryption;
- protocol negotiation;
- replay protection;
- bounded streams;
- stateless validation where appropriate;
- rate limits;
- fuzzing.

---

## 17. Scheduler attacks

A compromised or malicious scheduler could:

- route sensitive work to colluding nodes;
- manipulate rewards;
- starve nodes;
- collect topology;
- force inefficient paths;
- accumulate model shards at an attacker.

Mitigations evolve over time:

- auditable placement decisions;
- signed assignments;
- separation of accounting;
- scheduler diversity;
- policy constraints;
- decentralized/federated scheduling research;
- privacy-preserving rendezvous.

The initial centralized scheduler is a trust concentration and must be documented as such.

---

## 18. Accounting fraud

Threats:

- forged receipts;
- duplicate claims;
- replayed work;
- fabricated performance;
- verifier collusion;
- Compute Credit manipulation.

Required properties:

```text
task binding
attempt binding
node binding
result binding
verification state
unique crediting
```

A receipt must not be creditable twice.

---

## 19. Compute Credit and future economic attacks

MORDECAI v0 uses internal, non-transferable Compute Credits rather than a cryptocurrency or blockchain token.

Even an internal credit system can be attacked through:

- Sybil farming;
- fabricated work;
- duplicate receipts;
- verifier collusion;
- artificial demand;
- accounting manipulation;
- denial-of-service for economic advantage.

Compute Credits must not be transferable between users in v0, and credit balances must not be treated as a cryptographic asset.

Any future transferable token, blockchain settlement mechanism, staking system, or exchange integration requires a separate threat model and legal/regulatory review.

Do not introduce transferable-token incentives before useful work can be measured and verified credibly.


---

## 20. Training poisoning

A malicious user or worker may attempt to insert poisoned experience into future training.

Threats:

- backdoors;
- targeted misinformation;
- benchmark gaming;
- malicious code examples;
- reward hacking.

Mitigations:

- provenance;
- filtering;
- deduplication;
- trust scoring;
- held-out evaluation;
- adversarial evaluation;
- candidate isolation;
- controlled promotion.

Production traffic must not directly update production weights.

---

## 21. Model promotion compromise

An attacker who controls promotion can replace the network's intelligence.

Promotion requires:

- immutable candidate IDs;
- reproducible evaluation metadata;
- authorization;
- audit trail;
- rollback.

Future governance may decentralize this process, but `everyone can push to production` is not decentralization. It is an incident.

---

## 22. Supply-chain attacks

Dependencies, containers, model files, build systems, and release artifacts are attack surfaces.

Mitigations:

- dependency review;
- lockfiles;
- reproducible builds where practical;
- signed releases;
- SBOMs;
- digest-pinned artifacts;
- CI hardening;
- minimal build permissions.

---

## 23. Privacy tiers

Conceptual tiers:

### Standard

Protect against passive transport observers and unnecessary worker knowledge where practical.

### Private

Add topology reduction, relays, padding/batching, and stronger job opacity.

### Dark Compute

Research target using secret sharing/MPC and stronger metadata resistance.

Do not advertise `Dark Compute` as production-secure before implementation and independent review.

---

## 24. Trust matrix

| Component | Initially trusted? | Long-term direction |
|---|---|---|
| User | No | No |
| Worker | No | No |
| Storage node | No | No |
| Relay | No | No |
| Gateway | Yes, partially | Minimize trust |
| Scheduler | Yes, initially | Reduce/federate trust |
| Verifier | Partial | Multiple/independent |
| Model registry | Yes, initially | Authenticated/auditable |
| Compute Credit accounting | Yes, initially | Auditable/verifiable |
| Training pipeline | Yes | Auditable and gated |

---

## 25. Security milestones

### PoC

- authenticated development nodes;
- encrypted transport;
- strict parsers;
- object digest verification;
- resource limits;
- failure-safe scheduling.

### Permissionless alpha

- node authentication;
- stronger sandboxing;
- verification;
- reputation;
- signed receipts;
- adversarial tests;
- fuzzing.

### Privacy alpha

- relays;
- topology hiding;
- task opacity;
- padding experiments;
- privacy measurement.

### Dark Compute research

- secret-shared tensors;
- MPC kernels;
- non-collusion scheduling;
- anonymous accounting.

---

## 26. What we do not claim

Early MORDECAI versions must not claim:

- complete anonymity;
- zero-knowledge inference;
- model confidentiality against colluding workers;
- Byzantine fault tolerance unless actually implemented;
- secure MPC merely because data was split;
- privacy merely because QUIC/TLS is used.

Precise claims beat impressive adjectives.

---

## 27. Security review checklist

For every major subsystem ask:

1. What is the asset?
2. Who is the adversary?
3. What does the adversary control?
4. What data crosses the trust boundary?
5. What is authenticated?
6. What is confidential?
7. What can be replayed?
8. What is bounded?
9. What happens on failure?
10. What gets logged?
11. Can two malicious parties collude?
12. Can an economic incentive make the attack profitable?

Update this document when the answers materially change.
