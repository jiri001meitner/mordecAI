# MORDECAI Security Policy

Security is a core architectural property of MORDECAI.

The network is designed to eventually execute work across independent and potentially malicious participants. Bugs affecting authentication, distributed execution, model artifacts, accounting, sandboxing, privacy, or protocol parsing may therefore have consequences beyond a conventional web application.

## Supported versions

MORDECAI is currently pre-release.

Until stable releases exist, security fixes are expected to target the current development branch and the latest published release, if any.

A formal supported-version matrix will be published with versioned releases.

## Reporting a vulnerability

**Do not open a public issue for an undisclosed vulnerability.**

Until a dedicated security mailbox or repository private-reporting mechanism is published, use the private security-reporting facility of the official source repository when available.

If no private channel is available, publish only a minimal request for a private contact channel without disclosing exploit details.

The project should establish a dedicated security contact before operating a public permissionless network.

## Include

A useful report contains:

- affected component;
- affected version/commit;
- vulnerability class;
- impact;
- reproduction steps;
- proof of concept where safe;
- required privileges;
- network position required;
- suggested mitigation, if known.

Do not include real user secrets or data unnecessarily.

## High-priority areas

Particularly important findings include:

- remote code execution;
- sandbox escape;
- authentication bypass;
- signature verification failure;
- private-key exposure;
- arbitrary model/object substitution;
- content-address verification bypass;
- accounting forgery;
- reward theft;
- validator bypass;
- peer identity spoofing;
- protocol parser memory-safety flaws;
- denial of service with low attacker cost;
- prompt/result disclosure across users;
- model-shard disclosure beyond intended design;
- secret-share leakage;
- topology deanonymization;
- training-data poisoning;
- unauthorized model promotion;
- supply-chain compromise.

## Threat assumptions

Do not assume that a network participant is honest.

Potential adversaries include:

- malicious workers;
- malicious users;
- Sybil operators;
- compromised nodes;
- colluding workers;
- malicious relays;
- malicious validators;
- compromised infrastructure;
- hostile model/data publishers.

See [`docs/THREAT_MODEL.md`](docs/THREAT_MODEL.md).

## Cryptography

MORDECAI does not treat custom cryptography as an acceptable shortcut.

Use reviewed implementations of established primitives.

Experimental privacy protocols must be explicitly identified as experimental and must document their assumptions.

## Secrets

Never report secrets in public issues or logs.

Examples:

- private keys;
- API credentials;
- wallet seeds;
- production database passwords;
- TLS private keys;
- MPC secret shares;
- authentication tokens.

If a secret is exposed, assume compromise and rotate it.

## Coordinated disclosure

The project intends to follow coordinated disclosure:

1. reporter privately provides sufficient details;
2. maintainers reproduce and assess;
3. a fix is developed and tested;
4. affected operators receive upgrade guidance where appropriate;
5. disclosure occurs after mitigation is available.

Exact timelines depend on severity and network impact.

## Safe research

Good-faith security research against infrastructure you own or are explicitly authorized to test is welcome.

Do not:

- access other users' data;
- disrupt public services unnecessarily;
- retain secrets obtained accidentally;
- use a vulnerability for economic extraction;
- attack third-party contributor nodes without authorization.

## Security is not a warranty

The project is experimental software and is provided under the warranty disclaimer in its license.

Security documentation describes intended engineering practice, not a guarantee that the software is free from vulnerabilities.
