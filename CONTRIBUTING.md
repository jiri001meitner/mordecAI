# Contributing to MORDECAI

Thank you for helping build MORDECAI.

MORDECAI is an ambitious distributed-systems and AI project. Contributions are welcome, but architectural discipline matters: we are building one shared intelligent system backed by heterogeneous, permissionless resources—not a generic GPU marketplace with an AI-shaped sticker attached.

Before contributing, read:

1. [`README.md`](README.md)
2. [`ARCHITECTURE.md`](ARCHITECTURE.md)
3. [`AGENTS.md`](AGENTS.md)
4. [`docs/PROTOCOL.md`](docs/PROTOCOL.md)
5. [`docs/THREAT_MODEL.md`](docs/THREAT_MODEL.md)

## Contribution principles

Prefer changes that are:

- small enough to review;
- measurable;
- testable;
- failure-aware;
- observable;
- protocol-versioned where required;
- explicit about security and privacy assumptions.

Do not silently redefine architectural invariants.

## Development priorities

Unless a task requires otherwise:

1. correctness;
2. testability;
3. fault tolerance;
4. observability;
5. security;
6. interoperability;
7. performance;
8. elegance.

## Before opening a change

Search existing issues and documentation.

For significant features, open a design discussion before implementing large amounts of code, especially for:

- wire protocol changes;
- scheduler semantics;
- accounting;
- cryptography;
- model partitioning;
- storage formats;
- identity;
- token/settlement logic;
- public APIs.

## Suggested workflow

```bash
git switch -c feature/my-change

# implement

# format
# lint
# test

git status
git diff
git commit
```

Exact build and test commands will be documented once the initial implementation stack is committed.

## Tests

Significant changes should include tests.

Distributed functionality should test failure paths, not only the happy path.

Relevant cases include:

- timeout;
- cancellation;
- duplicate delivery;
- stale worker state;
- corrupted object;
- worker disappearance;
- retry;
- incompatible protocol version;
- partial response;
- malformed peer input.

Never claim a test passed unless it was actually run.

If required hardware is unavailable, state that explicitly.

## Benchmarks

Performance changes require reproducible measurements.

Record at least:

- commit;
- hardware;
- model;
- precision;
- topology;
- node count;
- RTT;
- bandwidth;
- batch size;
- workload;
- relevant runtime settings.

`Faster` without a baseline is marketing, not a benchmark.

## Protocol changes

Wire-level changes require:

- explicit schema/version handling;
- compatibility analysis;
- updated protocol documentation;
- parser/serialization tests;
- malformed-input tests.

Do not assume all nodes upgrade simultaneously.

## Security-sensitive changes

For authentication, cryptography, sandboxing, privacy, identity, accounting, or untrusted-input processing, document:

- threat being addressed;
- trust assumptions;
- information exposed;
- failure behavior;
- relevant external protocol/library.

Do not invent cryptographic primitives.

## Dependencies

Before adding a dependency, consider:

- maintenance;
- security;
- license;
- transitive dependency cost;
- supported platforms;
- whether it is actually necessary.

## Models and datasets

Do not commit large model weights or datasets directly to Git.

Model and dataset contributions must have clear provenance and licensing.

Training data must not be created from user data without an explicit applicable policy and authorization.

## Commit messages

Prefer focused conventional-style messages:

```text
feat(node): add capability advertisement
feat(storage): verify content digest
fix(scheduler): handle stale worker placement
test(network): simulate interrupted stream
docs(protocol): define handshake v0
```

## Pull requests

A useful pull request explains:

- what changed;
- why;
- architectural impact;
- security/privacy impact;
- tests run;
- benchmarks, if relevant;
- known limitations.

Avoid unrelated formatting or refactoring in the same change.

## AI-assisted contributions

AI coding tools are welcome.

The contributor remains responsible for:

- correctness;
- licensing;
- tests;
- security;
- understanding submitted code.

AI-generated confidence is not a substitute for verification.

Repository agents must also follow [`AGENTS.md`](AGENTS.md).

## Licensing of contributions

Unless explicitly stated otherwise for a specific component, source-code contributions are submitted under the repository's GNU Affero General Public License, version 3 or later (`AGPL-3.0-or-later`).

By submitting a contribution, you represent that you have the right to submit it under the applicable project license.

### Future dual licensing

The project may later evaluate a Contributor License Agreement (CLA) or another contribution-rights model if dual licensing becomes a concrete requirement.

No CLA is imposed by this document today.

Do not assume that accepting an AGPL contribution automatically gives the project owner the right to relicense that contributor's copyright under arbitrary proprietary terms.

## Code of conduct

Treat other contributors professionally.

Technical disagreement is expected. Personal attacks, harassment, threats, or deliberate sabotage are not acceptable.

A dedicated `CODE_OF_CONDUCT.md` may be added as the community grows.

## Questions

For architecture questions, start with `ARCHITECTURE.md`.

For protocol questions, start with `docs/PROTOCOL.md`.

For security questions that could reveal a vulnerability, follow `SECURITY.md` rather than opening a public issue.
