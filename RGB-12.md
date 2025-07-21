---
Author: Maxim Orlovsky
Proposed: 24 May 2025
Revision: 1; 24 May 2025
License: CC0
---

# RGB consensus roadmap

`RGB-I-0` is the first RGB consensus version recommended for the production.

It is implemented for the first time in `rgb-core` library `v0.12.0`.

The future planned releases and their functionality is listed below.

## RGB-I-1

This version will bring support for the fallback seals (see RCP-240406A for the details).

The planned date for the release is the end of 2025-beginning of 2026.

## RGB-I-2

This version will bring zk-prover and verification support.

## RGB-I-3

This version will bring cross-contract method calls.

## Future consensus features

The following list of consensus-level features is not yet assigned to a specific release,
and may become part of the releases listed above:
- zk-AluVM instructions for the introspection of Bitcoin blockchain;
- zk-AluVM instructions accessing the state of the Lightning channels.

### RGB-II

There are no plans for a consensus-breaking future version of RGB
(which would have to receive "RGB-II" version number).
