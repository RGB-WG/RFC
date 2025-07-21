---
Author: Maxim Orlovsky
Proposed: 24 May 2025
Revision: 1; 24 May 2025
License: Unlicensed
---

# Versioning of RGB releases

This standard introduces a special numbering for RGB consensus and standard library ABI,
which extends the normal semantic API version numbering under semantic versioning v2.

## Motivation

Client-side validation and distributed systems requirements for backward compatibility
differs from the requirements of API versioning in usual software.

This standard was developed to account for these facts in
naming specific RGB releases in a systemic way.

## Specification

### Consensus versioning

RGB consensus releases are numbered by a pair of dash-separated numbers, following a `RGB-` prefix.

The first of the numbers in the pair uses a Roman numeral format and specifies "major version,"
which must increase with each consensus-breaking release
(a "pushback" in terms of client-side validation).

The second of the numbers in the pair uses an Arabic numeral format and specifies "minor version,"
which must increase with each backward-compatible consensus release
(a "fast-forward" in terms of client-side validation).

### Standard library ABI versioning

RGB standard library ABI releases are versioned according to semantic versioning v2.0,
with the following changes:
- only major and minor version numbers are used
  (i.e. no patch number, no suffixes and no build metadata);
- instead of a dot, the components are separated by a dash.

The ABI version MUST be always prefixed with the consensus version,
being separated from it by a dot.

Each new breaking (pushback) consensus release,
increasing the roman major version number of the consensus,
resets standard library ABI to start from `1-0`.

### Use with crate versioning

The consensus-level libraries `ultrasonic` and `rgb-core` starting from v0.12 release
MUST append the consensus release version to the version numer using `+` build metadata.

For instance,

```toml
[package]
version = "0.12.0+RGB-I-0"
```

The standard libraries residing in `sonic` and `rgb-std` repositories
MUST append both consensus release version and standard library ABI version to the version number
using `+` build metadata, separated by a dot:

```toml
[package]
version = "0.12.0+RGB-I-0.1-0"
```

### Start of numbering

Pre-`v0.12` releases must be numbered as `RGB-0-N` in the consensus
(with `N` replaced by a corresponding `rgb-core` minor version number, like `RGB-0-11`),
and as `0-N` in the standard library (for instance, `RGB-0-11.0-11`).

Version 0.12 release of `ultrasonic` and `rgb-core` is assigned consensus version number `RGB-I-0`.

Version 0.12 release of standard libraries is assigned ABI version number `1-0`.


## Examples

`RGB-I-0`: the initial RGB version released for production purposes.
`RGB-I-1`: the first release of RGB consensus which is planned to bring fallback seals.
`RGB-II-0`: the future "second" version of RGB (not planned at this moment).
`RGB-I-0.1.0`: the initial RGB standard library ABI version.

## Rationale

### Not using semantic versioning for consensus

Consensus does not match the semantic versioning requirements:
in client-side validation some breaking change in terms of semantic versioning
may be non-breaking.

For instance, if something was invalid, and become valid,
for the client-side validation it is a backward-compatible change,
while for the semantic version it is a breaking change.

### Use of Roman numerals

Since RGB consensus doesn't use semantic versioning,
using Roman numerals helps to distinguish it from other semver-based versioning.

Roman numeration also helps to distinguish software releases

### Use of dash for separating version components

The dot, according to the semantic versioning v2, 
must be used to separate individual build metadata.
Thus, we use it to separate the consensus version from the standard library ABI version.
The only remaining valid separator character is dash,
which we use to separate components within the version number itself.

### Separate consensus and standard library ABI versioning

Under single consensus version there can be multiple standard library releases,
which may upgrade or break ABI without affecting the consensus.

### Separate standard library ABI and API versioning

Some standard library releases may not introduce breaking changes to the ABI,
but still break ABI compatibility.

Likewise, some API-breaking standard library changes (like chane it method arguments),
requiring a major version increase for the library, may not affect ABI at all.
