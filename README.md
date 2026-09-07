# MORDECAI

> **One intelligence. Powered by everyone.**

MORDECAI is a permissionless distributed AI network designed to run a shared intelligent system on resources contributed by people around the world.

Users consume **intelligence**, not rented hardware. Contributors provide CPU, GPU, NPU, RAM, storage, and network capacity. MORDECAI combines those resources into an execution fabric capable of running models and AI workloads that may be too large for any individual participant to operate alone.

The long-term goal is simple to describe and difficult to build:

> **A global AI that exists because the network exists.**

---

## What is MORDECAI?

MORDECAI combines several ideas into one system:

- one shared AI rather than a marketplace of unrelated models;
- a large distributed Core model;
- smaller specialist models and deterministic tools;
- hierarchical planning and task delegation;
- heterogeneous compute contributed by independent nodes;
- distributed model storage, memory, and caching;
- useful-work verification and contributor rewards;
- privacy-oriented execution;
- controlled continual learning from real work.

A user should never need to know whether a response used a GPU in Prague, an NPU in a phone, a RAM cache in Frankfurt, or twenty machines solving independent branches of the same problem.

They ask MORDECAI.

MORDECAI decides how to compute the answer.

---

## The idea

Traditional AI infrastructure looks roughly like this:

```text
Users
  │
  ▼
AI Provider
  │
  ▼
Datacenter
  │
  ▼
Large GPU Cluster
```

MORDECAI aims for:

```text
                         Users
                           │
                           ▼
                    ┌──────────────┐
                    │   MORDECAI   │
                    │ Intelligence │
                    └──────┬───────┘
                           │
                  planning / routing
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
           Core         Experts        Tools
             │             │             │
             └─────────────┼─────────────┘
                           │
                           ▼
                 Distributed Fabric
                           │
        ┌─────────┬────────┼────────┬─────────┐
        ▼         ▼        ▼        ▼         ▼
       CPU       GPU      NPU      RAM       SSD
        │         │        │        │         │
        └─────────┴────────┴────────┴─────────┘
                    contributed nodes
```

The distributed machine is an implementation detail.

**The product is the AI.**

---

## One AI

MORDECAI is not intended to become a decentralized catalog where every contributor hosts an independent chatbot.

The network presents a common intelligent system.

At its center is **MORDECAI Core**, responsible for:

- understanding requests;
- reasoning and planning;
- decomposing complex tasks;
- selecting specialist models and tools;
- allocating inference-time compute;
- launching parallel solution attempts;
- verifying intermediate results;
- synthesizing the final response.

The Core itself may be physically distributed across many machines.

No individual contributor needs to possess the complete model.

---

## Intelligence hierarchy

Using the largest model for every operation would be wasteful.

MORDECAI can delegate work:

```text
                       MORDECAI CORE
                            │
        ┌──────────┬────────┼────────┬──────────┐
        ▼          ▼        ▼        ▼          ▼
      Math        Code   Research  Vision    Language
        │          │        │        │          │
        ▼          ▼        ▼        ▼          ▼
      Solver    Compiler  Retrieval  CV       Translator
      CAS       Tests     Search     OCR      Classifier
```

A difficult request may become a graph of smaller tasks.

Multiple solvers can work independently on the same problem and MORDECAI can compare their results before producing an answer.

This makes inference-time compute a controllable resource rather than simply increasing the size of one forward pass.

---

## Models larger than one machine

A major design goal is the ability to operate a model that no individual contributor can run alone.

Model state may be distributed using techniques such as:

- pipeline parallelism;
- Mixture-of-Experts placement;
- local tensor parallelism inside fast clusters;
- distributed KV caches;
- model shard replication;
- hierarchical VRAM/RAM/NVMe storage.

Example:

```text
request
   │
   ▼
┌─────────┐
│ Stage A │  Node / cluster 1
└────┬────┘
     │ activations
     ▼
┌─────────┐
│ Stage B │  Node / cluster 2
└────┬────┘
     │
     ▼
┌─────────┐
│ Expert  │  Node / cluster 3
└────┬────┘
     │
     ▼
┌─────────┐
│ Stage C │  Node / cluster 4
└────┬────┘
     │
     ▼
 response
```

MORDECAI is WAN-aware. Fine-grained communication across arbitrary Internet links is expensive, so execution should favor locality and larger computational stages where possible.

---

## Anyone can contribute

A MORDECAI node may contribute any useful combination of:

- **CPU** — preprocessing, routing, retrieval, verification, symbolic work;
- **GPU** — tensor computation and model inference/training;
- **NPU** — efficient edge inference, embeddings, classifiers, small experts;
- **RAM** — model shards, caches, intermediate state;
- **SSD/NVMe** — models, checkpoints, datasets, distributed object storage;
- **bandwidth** — relay, model distribution, cache transfer;
- **cryptographic compute** — verification and privacy protocols.

A node might be:

- a phone;
- a laptop;
- a gaming PC;
- a workstation;
- a server;
- an eight-GPU machine;
- a rack;
- an entire datacenter.

The scheduler cares about capabilities, locality, reliability, and current availability—not whether the hardware has a sufficiently expensive logo on the front.

---

## Mobile devices

Phones are not second-class nodes.

Modern mobile devices contain surprisingly capable NPUs and highly energy-efficient compute.

MORDECAI may use them for workloads such as:

- embeddings;
- reranking;
- lightweight experts;
- speculative decoding;
- classification;
- image preprocessing;
- audio processing;
- validation;
- encrypted relay traffic.

A mobile node should support policies such as:

```text
✓ contribute only while charging
✓ Wi-Fi only
✓ minimum battery level
✓ temperature limit
✓ storage limit
✓ bandwidth limit
✓ idle only
```

A billion small accelerators are an interesting machine.

---

## Distributed storage

MORDECAI uses an object-oriented distributed storage model rather than pretending that Internet latency is a particularly unfortunate DIMM.

Objects may include:

- model shards;
- experts;
- checkpoints;
- datasets;
- embeddings;
- task artifacts;
- experience records;
- runtime images.

Objects are content-addressed:

```text
mordecai://blake3/<object-digest>
```

The storage layer can use replication, erasure coding, geographic placement, caches, and periodic verification.

A human-readable namespace may map onto immutable objects:

```text
/ai/models/core/
/ai/models/experts/
/ai/checkpoints/
/ai/knowledge/
/ai/experience/
/ai/evaluation/
```

---

## Memory hierarchy

Remote RAM is not local RAM.

MORDECAI therefore treats memory as a hierarchy:

```text
L0  accelerator memory
L1  local system RAM
L2  same-site / LAN RAM
L3  regional RAM cache
L4  distributed NVMe
L5  durable object storage
```

Scheduling, prefetching, caching, and data locality are fundamental parts of the runtime.

---

## Privacy

MORDECAI is designed for a hostile environment in which workers cannot automatically be trusted.

The long-term privacy goal is:

> **No individual contributor should have enough information to reconstruct the complete prompt, response, or model.**

Planned privacy mechanisms include:

- encrypted transport;
- pseudonymous nodes;
- model sharding;
- opaque job identifiers;
- relay routing;
- topology hiding;
- traffic padding and batching;
- secret sharing;
- multi-party computation;
- anonymous accounting;
- zero-knowledge techniques where useful.

Advanced privacy is a research track and should not prevent development of a functional distributed inference system first.

---

## Useful work

MORDECAI is not proof-of-work mining with an AI sticker glued to the case.

Contributors should ultimately receive Compute Credits for **verified useful contribution**.

**Important:** general trustless verification of heterogeneous neural-network computation is an open research problem. Early MORDECAI versions therefore use trusted development nodes and limited verification methods rather than claiming this problem is solved.

Verification may include:

- redundant execution;
- challenge tasks;
- deterministic validation;
- statistical validation;
- reputation;
- cryptographic receipts;
- storage proofs.

Simply claiming to own an H100 does not constitute work.

The network rewards results and useful availability.

---

## Compute Credits

MORDECAI v0 uses **Compute Credits**, an internal, non-transferable accounting unit.

Compute Credits are used to measure two things:

- verified contribution supplied to the network;
- AI services consumed from the network.

```text
verified contribution
        │
        ▼
    work receipt
        │
        ▼
 Compute Credits
        │
        ▼
     AI usage
```

A contributor may earn Compute Credits by providing useful work or resources accepted by the network. A user spends Compute Credits when consuming MORDECAI services.

For the initial system, Compute Credits are deliberately simple:

- they are stored in MORDECAI's internal accounting system;
- they are not implemented as a cryptocurrency;
- they are not blockchain tokens;
- they are not transferable between users;
- they are not designed for trading or exchange;
- they do not require a wallet.

The purpose of the credit system is to measure and test the real economics of distributed AI before introducing a transferable asset.

Any future token, blockchain settlement mechanism, or externally transferable economic asset is a separate research, legal, and regulatory design decision.


---

## Blockchain and transferable assets

Blockchain is **not part of MORDECAI v0**.

The initial system uses internal Compute Credits and conventional accounting. No blockchain operation is required for inference, scheduling, object transfer, work receipts, or credit balances.

A future settlement layer may be researched only after:

- distributed inference works reliably;
- useful work can be measured;
- verification behavior is understood;
- the credit economy has real operating data;
- legal and regulatory requirements have been reviewed.

Even if a blockchain is introduced later, it must remain outside the inference datapath.


---

## MORDECAI learns

The network should improve from the work it performs.

That does **not** mean immediately backpropagating every conversation into the production model.

The intended loop is:

```text
work
 │
 ▼
experience
 │
 ▼
filter / select
 │
 ▼
training
 │
 ▼
candidate models
 │
 ▼
evaluation
 │
 ├── reject
 │
 └── promote
       │
       ▼
   production
```

MORDECAI can retain:

- successful strategies;
- failed strategies;
- decompositions;
- verifier outcomes;
- benchmark results;
- tool usage;
- execution cost;
- feedback where permitted.

Specialists may evolve faster than the Core.

Core releases should be evaluated, versioned, reproducible, and reversible.

---

## Failure is normal

Permissionless nodes disappear.

Laptops suspend. Phones leave Wi-Fi. Processes crash. Networks partition. Somebody eventually trips over a fiber.

MORDECAI assumes failure rather than treating it as an exceptional event.

The runtime should support:

- replicas;
- retries;
- speculative execution;
- checkpoints;
- shard recovery;
- session migration;
- degraded execution;
- worker reputation.

A healthy distributed system is one that continues operating while individual pieces are misbehaving.

---

## What MORDECAI is not

MORDECAI is not primarily:

- a GPU rental marketplace;
- a generic decentralized cloud;
- a giant remote Linux VM;
- a Kubernetes replacement;
- a cryptocurrency mining pool;
- blockchain-hosted AI;
- a directory of unrelated chatbots.

The resource network exists for one reason:

**to operate and improve MORDECAI.**

---

## Architecture

The architecture is described in detail in [`ARCHITECTURE.md`](ARCHITECTURE.md).

At the highest level:

```text
USER
 │
 ▼
INTELLIGENCE API
 │
 ▼
MORDECAI CORE
 │
 ▼
ORCHESTRATION
 │
 ▼
MODELS / AGENTS / TOOLS
 │
 ▼
DISTRIBUTED EXECUTION FABRIC
 │
 ▼
CPU / GPU / NPU / RAM / STORAGE / NETWORK
```

---

## Development roadmap

### Phase 0 — Prototype

- gateway;
- node daemon;
- capability discovery;
- scheduler;
- object storage;
- basic accounting;
- dashboard.

### Phase 1 — Distributed model

Run one model across multiple independent Internet-connected workers.

Demonstrate:

- model sharding;
- pipeline inference;
- worker failover;
- model shard caching;
- OpenAI-compatible API.

### Phase 2 — Orchestration

Add:

- specialist models;
- tools;
- task DAGs;
- parallel solvers;
- verification;
- Core synthesis.

### Phase 3 — Permissionless network

Add:

- public node software;
- automatic benchmarks;
- resource discovery;
- work verification;
- reputation;
- contributor accounting.

### Phase 4 — Heterogeneous fabric

Add:

- CPU-only nodes;
- RAM/cache nodes;
- storage nodes;
- NPU/mobile nodes;
- bandwidth relays.

### Phase 5 — Learning

Add:

- experience storage;
- distributed training;
- candidate model lifecycle;
- distributed evaluation;
- controlled model promotion.

### Phase 6 — Advanced privacy

Research and deploy:

- topology-hiding overlays;
- metadata resistance;
- secret sharing;
- MPC compute cells;
- anonymous accounting;
- privacy-preserving model execution.

---

## Current status

**MORDECAI is currently an architecture and research project.**

The protocol, Compute Credit accounting, privacy system, model architecture, verification methods, and implementation details are under active design.

Nothing described here should be interpreted as a claim that the complete network already exists.

The initial engineering priority is intentionally less glamorous:

> Make multiple machines on the Internet reliably behave like one AI.

Once that works, we can argue with Byzantine nodes, cryptography, economics, mobile NPUs, and the rest of the distributed-systems zoo.

---

## Principles

1. **The user buys intelligence, not hardware.**
2. **Anyone should be able to contribute useful resources.**
3. **The Core may be larger than any individual participant.**
4. **Heterogeneous hardware is normal.**
5. **Useful work must be verified.**
6. **Privacy should come from architecture, not promises alone.**
7. **Node failure is expected.**
8. **Learning must be evaluated before reaching production.**
9. **Blockchain stays outside the inference datapath.**
10. **The network exists to create one shared intelligence.**

---

## Contributing

MORDECAI will need work across many areas:

- distributed systems;
- Linux;
- networking;
- machine learning;
- inference runtimes;
- model parallelism;
- cryptography;
- storage;
- schedulers;
- mobile/edge compute;
- economics;
- security;
- observability.

Contribution guidelines and development environment documentation will be added as the implementation takes shape.

---

## License

License has not yet been selected.

The project is intended to have a strong open-source component, but the exact licensing model for the protocol, node software, model code, weights, and related components must be defined separately.

---

## Name

**MORDECAI**

The name can also be read typographically as:

**MORDEC·AI**

Project website: `mordecai.eu`

---

**MORDECAI — One intelligence. Powered by everyone.**
