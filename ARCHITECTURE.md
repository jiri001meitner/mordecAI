# MORDECAI Architecture

> **MORDECAI** — One intelligence. Powered by everyone.

MORDECAI is a permissionless distributed AI network designed to operate a shared intelligent system across heterogeneous hardware contributed by independent participants.

The public product is **AI**, not infrastructure rental.

Users interact with one AI system through chat, research, coding, analysis, creation, and API interfaces. Under the hood, MORDECAI assembles CPU, GPU, NPU, RAM, storage, and network resources from many contributors into a distributed execution fabric capable of hosting models that may be too large for any single participant to run alone.

The long-term goal is a shared, continuously improving intelligence whose compute, memory, storage, orchestration, and learning infrastructure are distributed across the network.

---

## 1. Design Goals

MORDECAI is designed around the following principles.

### 1.1 One AI, not a GPU marketplace

Users do not request:

- 2× H100
- 128 GB RAM
- 4 TB NVMe
- 12 CPU cores

Users request intelligence:

- answer a question
- perform research
- write code
- analyze data
- generate media
- solve a complex task
- call an API

The scheduler and orchestration layers decide how much infrastructure is required.

### 1.2 Permissionless participation

Anyone may contribute compatible resources:

- CPU
- GPU
- mobile or desktop NPU
- RAM
- persistent storage
- network bandwidth
- cryptographic compute
- combinations of the above

A contributor may provide a phone, laptop, workstation, server, rack, or datacenter cluster.

### 1.3 Models larger than any individual participant

The core AI should not require any single node to possess the complete model.

Weights, experts, caches, checkpoints, and runtime state may be sharded across multiple nodes.

### 1.4 Heterogeneous by design

The network must assume:

- different architectures
- different accelerator vendors
- different memory sizes
- changing node availability
- variable network latency
- variable bandwidth
- unreliable devices
- mobile devices entering and leaving frequently

Homogeneous clusters are an optimization, not an assumption.

### 1.5 Privacy by architecture

The system should minimize how much any participant can learn about:

- user identity
- prompt contents
- model identity
- model weights
- job purpose
- output
- neighboring workers
- global topology

No individual contributor should need enough information to reconstruct the complete prompt, response, or model.

### 1.6 Useful-work economy

Users spend internal MORDECAI Compute Credits to consume intelligence.

Contributors earn Compute Credits by providing useful work or resources accepted by the network.

A user may therefore be:

- a net consumer
- approximately neutral
- a net contributor

The economy exists to allocate AI capacity, not to turn the system into a hardware marketplace.

For v0, Compute Credits are internal, non-transferable accounting units. They are not cryptocurrency, are not blockchain tokens, are not user-to-user transferable, and do not require wallets. Any future transferable asset or blockchain settlement mechanism is outside the v0 architecture and requires separate technical, economic, legal, and regulatory design.

### 1.7 Continuous improvement without uncontrolled self-modification

MORDECAI should learn from completed work, but production models must not directly rewrite themselves from arbitrary user traffic.

Learning is gated through:

1. experience capture
2. data filtering
3. training
4. evaluation
5. candidate comparison
6. controlled promotion
7. rollback capability

---

# 2. High-Level Architecture

```text
                           ┌───────────────────────────┐
                           │        USER LAYER         │
                           │ Chat / API / Research /   │
                           │ Code / Analyze / Create   │
                           └─────────────┬─────────────┘
                                         │
                                         ▼
                           ┌───────────────────────────┐
                           │     INTELLIGENCE GATEWAY  │
                           │ auth / budget / policy /  │
                           │ routing / privacy ingress │
                           └─────────────┬─────────────┘
                                         │
                                         ▼
                           ┌───────────────────────────┐
                           │      MORDECAI CORE        │
                           │ planning / reasoning /    │
                           │ decomposition / synthesis │
                           └─────────────┬─────────────┘
                                         │
                      ┌──────────────────┼──────────────────┐
                      │                  │                  │
                      ▼                  ▼                  ▼
               ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
               │ Specialist  │   │ Tool Agents │   │ Parallel    │
               │ Models      │   │ / Solvers   │   │ Solvers     │
               └──────┬──────┘   └──────┬──────┘   └──────┬──────┘
                      └──────────────────┼──────────────────┘
                                         │
                                         ▼
                           ┌───────────────────────────┐
                           │      EXECUTION FABRIC     │
                           │ scheduler / tensors /     │
                           │ memory / storage / streams│
                           └─────────────┬─────────────┘
                                         │
               ┌────────────┬────────────┼────────────┬────────────┐
               ▼            ▼            ▼            ▼            ▼
             CPU          GPU           NPU          RAM          SSD
             nodes        nodes         nodes        nodes        nodes
```

---

# 3. System Layers

MORDECAI is divided into logical layers. Implementations may combine components during early development, but the boundaries should remain explicit.

## 3.1 User Layer

The primary interfaces expose intelligence rather than infrastructure.

Initial interfaces:

- web chat
- OpenAI-compatible API
- research mode
- coding mode
- analysis mode
- task API
- contribution dashboard

Possible user-selectable execution profiles:

- Standard
- Deep
- Maximum

These profiles represent inference-time budget, not specific hardware.

---

## 3.2 Intelligence Gateway

The gateway accepts requests and converts them into internal work.

Responsibilities:

- API compatibility
- user authentication
- anonymous or pseudonymous session handling
- quota and Compute Credit balance
- maximum task budget
- request classification
- safety and policy enforcement
- privacy-tier negotiation
- request encryption
- ingress routing
- streaming output aggregation

The gateway must not expose physical worker topology to users.

---

# 4. MORDECAI Core

The Core is the primary reasoning system.

It is responsible for:

- understanding user intent
- planning
- task decomposition
- deciding which subtasks require specialists
- deciding which subtasks require deterministic tools
- allocating inference-time compute
- requesting parallel solution attempts
- comparing candidate answers
- synthesizing the final result

The Core is conceptually one AI, even when its model state is physically distributed.

## 4.1 Core model placement

The Core may use a combination of:

- pipeline parallelism
- expert parallelism
- local tensor parallelism inside fast clusters
- distributed KV-cache placement
- distributed model shard caching
- host RAM / accelerator VRAM hierarchy

Fine-grained tensor parallelism over arbitrary WAN links should be avoided wherever possible.

WAN-aware execution should favor larger computational stages with less communication.

---

# 5. Hierarchical Intelligence

MORDECAI should not spend its largest model on every arithmetic operation.

The Core delegates work to specialized systems.

Example topology:

```text
                           MORDECAI CORE
                                │
          ┌───────────┬─────────┼─────────┬───────────┐
          │           │         │         │           │
          ▼           ▼         ▼         ▼           ▼
        Math         Code    Research   Vision     Language
          │           │         │         │           │
          ▼           ▼         ▼         ▼           ▼
      verifier     compiler    retrieval   CV        translation
      CAS          tests       search      OCR       speech
      solver       fuzzing     ranking     media     classifiers
```

Specialists may be:

- smaller language models
- MoE experts
- embedding models
- rerankers
- image models
- speech models
- symbolic systems
- deterministic programs
- external tools
- internally trained domain agents

The orchestrator may launch multiple independent solvers for the same problem.

## 5.1 Skills layer

Skills represent reusable capabilities available to the Core and orchestrator.

Examples include:

- research workflows;
- code-analysis procedures;
- data transformation;
- deterministic tool chains;
- domain-specific execution recipes;
- specialist invocation patterns.

A skill should define its inputs, outputs, requirements, permissions, and failure behavior.

Skills are orchestration/runtime definitions, not a mechanism for bypassing the scheduler or sandbox. Installing or retrieving a skill must not automatically grant it arbitrary code execution or access to contributor hosts.

Repository-managed skill definitions live under `skills/`.

## 5.2 Knowledge layer

Knowledge represents information MORDECAI can retrieve and use without forcing every fact into model weights.

The knowledge layer may contain or reference:

- documents;
- structured knowledge;
- retrieval sources;
- indexes;
- embeddings;
- provenance;
- source metadata;
- ingestion definitions.

Repository-managed definitions and small static assets live under `knowledge/`.

Large datasets, embeddings, vector indexes, private user material, and mutable runtime state should live in the storage layer rather than Git. The repository should reference those objects through manifests or content-addressed identifiers where practical.

Knowledge is untrusted input unless explicitly classified otherwise. Retrieved content cannot grant itself tool permissions, execute arbitrary code, or override higher-level policy.

Conceptually:

```text
                       MORDECAI CORE
                            │
          ┌─────────┬───────┼──────────┬─────────┐
          ▼         ▼       ▼          ▼         ▼
     Specialists  Skills  Knowledge   Tools   Solvers
          │         │       │          │         │
          └─────────┴───────┼──────────┴─────────┘
                            ▼
                         Task DAG
                            │
                            ▼
                     Execution Fabric
```


---

# 6. Task Graph

Complex user requests are represented as a DAG.

Example:

```text
User request
    │
    ▼
Understand task
    │
    ▼
Create plan
    │
    ├──────────────┬──────────────┬──────────────┐
    ▼              ▼              ▼              ▼
Research        Math model      Code PoC       Critic
    │              │              │              │
    └──────────────┴──────┬───────┴──────────────┘
                          ▼
                     Verification
                          │
                          ▼
                       Synthesis
                          │
                          ▼
                        Answer
```

A task node should contain:

- opaque task ID
- operation class
- dependency IDs
- resource requirements
- privacy requirements
- deadline
- maximum cost
- retry policy
- verification policy
- result hash

---

# 7. Distributed Model Architecture

## 7.1 Weight sharding

No requirement exists for any worker to hold the complete model.

Model artifacts are split into content-addressed shards.

Possible partitioning strategies:

- layer groups
- attention blocks
- MoE experts
- embedding layers
- output layers
- auxiliary model components

Each shard may have multiple replicas.

## 7.2 Mixture of Experts

MoE is particularly attractive for MORDECAI because:

- total parameter count may greatly exceed active parameters per token
- experts can be distributed independently
- frequently used experts can have more replicas
- specialist nodes may host specific expert families
- expert placement can follow geography and hardware type

Routing itself must avoid leaking semantic information where privacy tiers require opacity.

## 7.3 Pipeline execution

A simplified inference path:

```text
tokens
  │
  ▼
Stage A
  │ activations
  ▼
Stage B
  │
  ▼
Expert cell
  │
  ▼
Stage C
  │
  ▼
Output head
```

Microbatching is used to increase throughput.

The scheduler should group compatible sessions where possible.

---

# 8. Resource Fabric

The distributed machine is an implementation detail of MORDECAI, not the user-facing product.

Resources are represented abstractly.

## 8.1 Resource classes

Initial resource classes:

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

Hardware maps into these classes.

Examples:

- x86 CPU → scalar_compute
- ARM CPU → scalar_compute
- NVIDIA GPU → tensor_compute + fast_memory
- AMD GPU → tensor_compute + fast_memory
- Apple Neural Engine → vector/tensor_compute
- Qualcomm NPU → vector/tensor_compute
- 1 TB RAM server → bulk_memory
- NVMe node → persistent_storage
- high-bandwidth relay → network_bandwidth

---

# 9. Node Architecture

A contributor runs a MORDECAI Node.

```text
┌─────────────────────────────────────────┐
│              MORDECAI NODE              │
├─────────────────────────────────────────┤
│ identity / reputation                   │
│ capability discovery                    │
│ resource policy                         │
│ job runtime                             │
│ sandbox                                 │
│ model/object cache                      │
│ encrypted transport                     │
│ benchmark agent                         │
│ telemetry                               │
│ accounting receipts                     │
└─────────────────────────────────────────┘
```

The node advertises capability, not user-facing hardware rental.

Example capability descriptor:

```yaml
node:
  architecture: x86_64

resources:
  scalar_compute:
    cores: 32

  tensor_compute:
    backend: cuda
    devices: 2
    precision:
      - fp16
      - bf16
      - int8

  fast_memory:
    bytes: 51539607552

  bulk_memory:
    bytes: 274877906944

  persistent_storage:
    bytes: 2199023255552

policy:
  max_power_watts: 650
  idle_only: false
  max_bandwidth_mbps: 500
```

---

# 10. Mobile Participation

Phones and tablets are first-class contributors.

Mobile contributions may include:

- NPU inference
- embeddings
- reranking
- lightweight experts
- speculative decoding
- image preprocessing
- audio feature extraction
- CPU preprocessing
- encrypted relay traffic
- local cache
- validation tasks

Recommended user controls:

- only while charging
- only on Wi-Fi
- minimum battery percentage
- pause above temperature threshold
- maximum storage contribution
- maximum bandwidth
- idle-only mode
- scheduled participation

Mobile devices should receive tasks suited to their energy and connectivity profile.

---

# 11. Memory Hierarchy

WAN memory is not RAM.

MORDECAI must use a latency-aware hierarchy.

```text
L0  Accelerator VRAM / local accelerator SRAM
L1  Local host RAM
L2  Same-site / same-LAN remote RAM
L3  Regional distributed RAM cache
L4  Distributed NVMe cache
L5  Durable object storage
```

The runtime should use:

- prefetching
- asynchronous staging
- replication
- locality-aware scheduling
- cache heat tracking
- eviction policies
- immutable content addressing

Remote memory should expose object/tensor semantics rather than transparent CPU load/store semantics.

---

# 12. Distributed Storage

MORDECAI should be object-first, not POSIX-first.

## 12.1 Content addressing

Objects are identified by cryptographic digest.

Example:

```text
mordecai://blake3/<digest>
```

Possible object types:

- model shard
- expert
- tokenizer
- checkpoint
- dataset chunk
- embedding block
- task artifact
- experience record
- evaluation artifact
- software runtime image

## 12.2 Reliability

Storage may use:

- replication
- erasure coding
- geographic distribution
- periodic possession verification
- object repair
- cache promotion

Illustrative erasure profile:

```text
10 data shards + 4 parity shards
```

This is a policy choice, not a fixed requirement.

## 12.3 Namespace

A human-readable virtual namespace may exist above immutable objects:

```text
/ai/models/core/
/ai/models/experts/
/ai/checkpoints/
/ai/tasks/
/ai/knowledge/
/ai/experience/
/ai/evaluation/
/runtime/
```

Internally, mutable namespace entries resolve to immutable object hashes.

---

# 13. Network Topology

MORDECAI should use an overlay network optimized for:

- NAT traversal
- encrypted point-to-point channels
- relay support
- capability discovery
- topology hiding
- locality optimization
- high-throughput tensor transport

QUIC is a strong candidate for many transport paths.

The system should distinguish:

- control traffic
- tensor/data traffic
- model distribution
- storage traffic
- accounting receipts
- privacy relays

---

# 14. Scheduler

The scheduler is one of the critical components of MORDECAI.

It creates execution topologies from available resources.

Inputs may include:

- operation type
- required memory
- compute backend
- precision
- expected FLOPS
- worker latency
- bandwidth
- data locality
- model shard locality
- reliability score
- availability history
- geography
- privacy policy
- expected execution time
- energy/cost score

The scheduler should prefer clusters with high local bandwidth for communication-heavy work.

Example:

```text
                   Global Scheduler
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      EU swarm        US swarm       APAC swarm
          │
     ┌────┴────┐
     ▼         ▼
 local GPU   RAM/cache
 cluster      nodes
```

A datacenter with eight connected GPUs may be treated as one logical worker.

---

# 15. Privacy Architecture

Privacy is a protocol property, not merely a terms-of-service promise.

MORDECAI should support multiple privacy tiers.

## 15.1 Standard

- encrypted transport
- pseudonymous workers
- model sharding
- minimal logs
- job isolation

## 15.2 Private

Adds:

- relay routing
- hidden worker relationships
- opaque job identifiers
- padding
- batching
- reduced metadata
- randomized scheduling

## 15.3 Dark Compute

Long-term target.

Adds:

- secret-shared activations
- secret-shared model state where practical
- MPC compute cells
- anonymous accounting
- traffic shaping
- cover traffic where required

Security goal:

> No individual contributor has enough information to reconstruct the prompt, response, or complete model.

---

# 16. Compute Cells

Sensitive operations may execute inside multi-party compute cells.

Example 3-party cell:

```text
                    encrypted input
                          │
                          ▼
                    share splitter
                    /      |      \
                   /       |       \
                  ▼        ▼        ▼
               Worker A Worker B Worker C
                  │        │        │
                  └──── MPC operation ────┘
                          │
                          ▼
                   shared output
```

For a tensor `X`:

```text
X = X1 + X2 + X3 mod p
```

Each individual share should reveal no useful information about `X`.

Multiplication requires MPC protocols such as preprocessed multiplication triples or equivalent techniques.

This layer is research-heavy and should not block the earliest PoC.

---

# 17. Threat Model

MORDECAI must assume some participants are malicious.

Threats include:

- incorrect computation
- fabricated resource claims
- Sybil nodes
- model shard theft
- prompt extraction
- output extraction
- worker collusion
- traffic analysis
- denial of service
- validator collusion
- accounting fraud
- poisoned training data
- malicious model updates
- storage corruption

No single mechanism solves all of these.

The architecture therefore combines:

- replication
- challenge work
- deterministic verification where possible
- probabilistic verification
- reputation
- economic penalties
- cryptographic receipts
- content addressing
- isolated training pipelines
- privacy routing
- threshold trust

---

# 18. Work Verification

Contributors are rewarded for verified useful work, not self-reported activity.

Possible verification strategies:

## 18.1 Redundant execution

Run selected jobs on independent nodes and compare outputs.

## 18.2 Challenge jobs

Inject known or partially known workloads.

## 18.3 Deterministic verification

Suitable for:

- compilation
- hashes
- symbolic math
- some preprocessing
- storage proofs

## 18.4 Statistical verification

Suitable for model inference where exact bitwise equality may not be reliable.

## 18.5 Reputation

Track long-term correctness and availability.

Reputation should not map trivially to a public wallet identity when privacy is required.

---

# 19. Accounting and Compute Credits

MORDECAI v0 uses internal **Compute Credits** to account for contribution and AI consumption.

A Compute Credit is an internal, non-transferable accounting unit. It is not a cryptocurrency or blockchain token.

Conceptually:

```text
useful work
    ↓
work receipt
    ↓
verification / confidence
    ↓
credit accounting
    ↓
Compute Credit balance
    ↓
AI consumption
```

The accounting system should record:

- task and attempt identity;
- contributor identity;
- measured resource usage;
- result digest;
- verification method and confidence;
- credited amount;
- user consumption;
- immutable audit history.

A work receipt is evidence used by accounting. A receipt is not automatically proof that arbitrary heterogeneous neural-network computation was correct.

## 19.1 Verification is an open research problem

General trustless verification of heterogeneous AI computation is **not solved by the architecture**.

Different workloads permit different verification methods:

- deterministic workloads may be recomputed exactly;
- replicated execution may compare independent results;
- challenge jobs may test selected capabilities;
- statistical checks may provide partial confidence;
- specialized verifiers may exist for specific operations.

GPU inference can vary across hardware, kernels, drivers, precision modes, and numerical implementations. Naive equality checks are therefore insufficient for many workloads, while tolerance-based comparison creates its own adversarial and economic questions.

Cryptographic proof systems and trusted hardware may be researched, but neither is assumed to provide a universal low-cost solution.

For the controlled prototype, trusted development nodes are acceptable. Permissionless rewards require stronger workload-specific verification.

## 19.2 No blockchain in v0

MORDECAI v0 does not require a blockchain, wallet, transferable token, exchange, staking system, or on-chain settlement.

A future settlement layer may be researched after the network has measured real distributed inference costs and developed credible useful-work verification.

Any future transferable economic asset requires separate technical, economic, legal, and regulatory review.


# 20. Off-Chain Execution Accounting

Per-job blockchain transactions would be too expensive and slow.

Execution accounting should occur off-chain.

Possible design:

```text
job
 │
 ▼
signed receipts
 │
 ▼
epoch accounting
 │
 ▼
Merkle root
 │
 ▼
settlement layer
```

The blockchain, if used, should handle:

- asset ownership
- periodic settlement
- staking/slashing
- governance
- protocol parameters

It should not transport:

- tensors
- prompts
- model files
- task graphs
- filesystem operations

---

# 21. Identity

Different identity scopes should exist.

## 21.1 User identity

May be:

- account-based
- wallet-based
- pseudonymous
- privacy-preserving credential based

## 21.2 Node identity

Used for:

- capability history
- reliability
- work receipts
- reputation

## 21.3 Economic identity

Should be separable from runtime identity where possible.

This prevents simple correlation between:

- who performed work
- what task was performed
- who was paid
- who later purchased AI usage

Future privacy mechanisms may include:

- blind signatures
- anonymous credentials
- zero-knowledge proofs

---

# 22. Continual Learning

MORDECAI should improve from real work without blindly training on all user content.

The learning pipeline is:

```text
Production tasks
      │
      ▼
Experience capture
      │
      ▼
Filtering / privacy / consent
      │
      ▼
Experience lake
      │
      ▼
Selection and curriculum
      │
      ▼
Training
      │
      ▼
Candidate models
      │
      ▼
Evaluation
      │
      ├──── reject
      │
      └──── promote
             │
             ▼
        Production
```

## 22.1 Experience records

Useful experience may include:

- original problem representation
- decomposition
- chosen tools
- attempted strategies
- failed approaches
- intermediate artifacts
- verifier results
- execution cost
- latency
- critic scores
- accepted answer
- user feedback where permitted

Failure data is valuable.

The system should learn not only what works, but what repeatedly wastes resources.

---

# 23. Memory Types

MORDECAI should distinguish three forms of memory.

## 23.1 Parametric memory

Knowledge encoded in model weights.

Slowest and most expensive to update.

## 23.2 Semantic memory

Explicit knowledge stores:

- documents
- embeddings
- structured knowledge
- verified facts
- code repositories
- indexes

Can update continuously.

## 23.3 Episodic memory

Experience from previous tasks:

- strategies
- outcomes
- tool traces
- benchmarks
- failures
- successful plans

This is central to continual improvement.

---

# 24. Training Strategy

Different components may train at different cadences.

Illustrative cadence:

```text
semantic memory        continuous
experience memory      continuous
light adapters         hours
specialist models      daily
core model             weeks/months
```

Training may use spare network capacity.

Inference should usually have priority over non-urgent training.

---

# 25. Model Evolution

Core updates should be evaluated competitively.

Example:

```text
                 Current Core v48
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
      v49-A         v49-B         v49-C
        │             │             │
        └──────── distributed eval ─┘
                      │
                      ▼
                   winner
                      │
                      ▼
                   v49
```

Evaluation should include:

- correctness
- reasoning
- coding
- safety
- latency
- token efficiency
- resource cost
- adversarial testing
- privacy regression
- distributed runtime behavior

Every production promotion requires rollback capability.

---

# 26. Failure Handling

Nodes are assumed to disappear.

The runtime must support:

- timeouts
- retries
- stage replication
- speculative execution
- checkpointing
- shard replicas
- recomputation
- session migration
- degraded execution

For critical model stages, maintain multiple candidate workers.

Example:

```text
          Stage N
             │
       ┌─────┴─────┐
       ▼           ▼
    primary      replica
       │           │
       └──── failover
```

---

# 27. Observability

Public dashboards may expose aggregate system state.

Possible metrics:

- active nodes
- countries/regions
- contributed CPU cores
- GPU VRAM
- NPU capacity
- RAM
- storage
- network bandwidth
- running inference tasks
- training activity
- active specialist models
- model version
- useful-work throughput
- average task latency
- verification rate

Privacy-sensitive topology must never be exposed directly.

---

# 28. Software Stack

The exact stack may evolve.

A pragmatic initial stack:

## Control plane

- Rust or Go
- gRPC and/or QUIC
- PostgreSQL
- NATS or equivalent messaging
- Prometheus
- Grafana

## AI runtime

- Python
- PyTorch
- CUDA / ROCm where available
- ONNX Runtime or vendor runtimes for edge/NPU
- containerized worker workloads

## Storage

- content-addressed object layer
- S3-compatible backing where useful
- local NVMe/RAM caches
- BLAKE3 or equivalent fast cryptographic hashing

## Node sandbox

Candidates include:

- OCI/containerd
- rootless containers
- WebAssembly for suitable workloads
- additional sandboxing for untrusted kernels

---

# 29. API Compatibility

The initial public API should emulate common AI APIs where practical.

Example:

```http
POST /v1/chat/completions
POST /v1/responses
POST /v1/embeddings
```

MORDECAI-specific extensions may include:

```json
{
  "mode": "deep",
  "max_budget": 100,
  "privacy": "private"
}
```

The public API must remain independent of physical execution topology.

---

# 30. MVP / PoC

The first version does not need to solve the entire research problem.

The goal is to prove the core architectural thesis.

## Phase 0 — Local prototype

Build:

- MORDECAI gateway
- node daemon
- capability discovery
- scheduler
- object store
- basic accounting
- dashboard

Run across a small controlled cluster.

## Phase 1 — Internet model sharding

Demonstrate one model whose weights do not fit on any individual worker.

Example target:

- 4–8 independent workers
- heterogeneous GPUs
- model shards distributed over Internet links
- pipeline inference
- worker failure recovery
- OpenAI-compatible API

## Phase 2 — Specialist orchestration

Add:

- math worker
- code worker
- retrieval agent
- parallel solvers
- critic/verifier
- Core synthesis

## Phase 3 — Permissionless contributors

Add:

- public node software
- automatic benchmarking
- capability registry
- job verification
- contributor accounting
- reputation

## Phase 4 — Heterogeneous resources

Add:

- CPU-only nodes
- RAM/cache nodes
- storage nodes
- mobile NPU workers
- bandwidth relays

## Phase 5 — Continual learning

Add:

- experience lake
- training jobs
- candidate model lifecycle
- distributed evaluation
- safe promotion

## Phase 6 — Advanced privacy

Add:

- relay topology
- hidden task identities
- traffic padding
- anonymous settlement
- MPC compute cells
- secret-shared execution experiments

---

# 31. Non-Goals

MORDECAI is not primarily:

- a GPU rental marketplace
- a generic decentralized cloud
- a distributed Linux VM
- a replacement for Kubernetes
- a cryptocurrency mining network
- a blockchain for storing AI workloads
- a collection of unrelated user-hosted chatbots

The distributed machine exists to operate one shared intelligent system.

---

# 32. Core Architectural Invariants

The following should remain true as the implementation evolves.

### Invariant 1

**The user buys intelligence, not infrastructure.**

### Invariant 2

**Contributors provide resources, not independently branded AI instances.**

### Invariant 3

**The Core can be physically larger than any individual node.**

### Invariant 4

**Hardware heterogeneity is normal.**

### Invariant 5

**No blockchain is required by MORDECAI v0, and no future blockchain may enter the inference datapath.**

### Invariant 6

**Production models do not directly self-modify from arbitrary user traffic.**

### Invariant 7

**A node is paid for verified useful contribution, not claimed capacity.**

### Invariant 8

**Privacy must improve through protocol design rather than operator promises alone.**

### Invariant 9

**Failure of individual nodes is expected and recoverable.**

### Invariant 10

**The distributed compute fabric is internal infrastructure; MORDECAI itself is an AI system.**

### Invariant 11

**Skills define capabilities and knowledge supplies information; neither automatically grants execution authority or bypasses runtime security boundaries.**

---

# 33. Open Research Problems

Several parts of the long-term design remain active research areas.

Most important:

1. Efficient transformer inference over heterogeneous WAN links.
2. Dynamic model partitioning under node churn.
3. WAN-efficient MoE expert placement.
4. Privacy-preserving expert routing.
5. Efficient MPC transformer operations.
6. Secret-shared KV cache.
7. Anonymous useful-work accounting.
8. Reliable verification of non-deterministic inference.
9. Secure continual learning from distributed experience.
10. Scheduling across GPU, NPU, CPU, RAM, storage, and network resources.
11. Incentive compatibility under adversarial participation.
12. Protecting model IP while maintaining a permissionless network.

MORDECAI should treat these as separable research tracks rather than blockers for the initial system.

---

# 34. Project Structure

A possible repository layout:

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
│   └── ARCHITECTURE.md
└── tests/
```

---

# 35. Architectural Summary

MORDECAI is a shared AI system backed by a permissionless global resource network.

Its architecture separates:

```text
INTELLIGENCE
    │
    ▼
ORCHESTRATION
    │
    ▼
MODEL EXECUTION
    │
    ▼
RESOURCE FABRIC
    │
    ▼
CPU / GPU / NPU / RAM / SSD / NETWORK
```

The infrastructure is distributed.

The intelligence is presented as one system.

The economy rewards useful contribution.

The learning system improves models through controlled evaluation.

The privacy architecture aims to prevent individual participants from reconstructing the user's work or the complete model.

The long-term objective is not merely to distribute computation.

It is to build an intelligence that can exist because the network exists.

---

## Status

This document describes the intended architecture and research direction of MORDECAI.

It is not a final protocol specification.

Early implementations should prioritize:

1. working inference,
2. model sharding,
3. failure tolerance,
4. scheduling,
5. useful-work accounting,

before attempting advanced cryptographic privacy or fully decentralized governance.
