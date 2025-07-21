---
Updated: 21 Jul 2025
---

# List of RGB repositories and libraries

In the lists below, the top level contains a reference to a repository,
and the layers underneath describe specific Rust crates with library and binary targets.
If the crates are not listed, than the repository contains a single crate named after it.

## Consensus layer

1. [LNP-BP/client_side_validation](https://github.com/LNP-BP/client_side_validation)
   core cryptographic primitives for client-side validation (not specific to RGB or Bitcoin)
   - `commit_verify`: implementation of tagged hashing, merklization, commitment creation workflows,
     multiprotocol commitments and multimessage bundles.
   - `commit_encoding_derive`: derivation macros for applying commitment workflows to data structures.
   - `single_use_seals`: single-use seal APIs.
   - `client_side_validation`: umbrella library combining the above.
2. [strict-types/strict_encoding](https://github.com/strict-types/strict_encoding)
   consensus data serialization procedures
   - `strict_encoding`
   - `strict_encoding_derive`
3. [AluVM/aluvm](https://github.com/AluVM/aluvm): framework for building virtual machines
4. [AluVM/zk-aluvm](https://github.com/AluVM/zk-aluvm): zk-compatible RISC virtual machine
5. [AluVM/ultrasonic](https://github.com/AluVM/ultrasonic): polynomial computer based on zk-AluVM
6. [BP-WG/bp-core](https://github.com/BP-WG/bp-core): consensus parts of bitcoin protocol
   - `bp-consensus`: Bitcoin blockchain consensus-level primitives,
   - `bp-dbc`: deterministic bitcoin commitments (opret, tapret),
   - `bp-seals`: bitcoin-specific single-use seals code
   - `bp-core`: umbrella library combining all of the above.
7. [RGB-WG/rgb-core](https://github.com/RGB-WG/rgb-core): main RGB consensus implementation
   orchestrating operations of the above libraries.

(see also [list of dependencies below](#external-dependencies-of-the-consensus-layer))

## Standard library

1. [UBIDECO/ascii-armor](https://github.com/UBIDECO/ascii-armor): ASCII armoring library,
   used for transforming binary RGB files into text - and vise versa.
2. [UBIDECO/rust-baid64](https://github.com/UBIDECO/rust-baid64): Base64-based encoding
   for RGB-related identifiers providing chunking, checksums, mnemonics etc.
3. [AluVM/sonic](https://github.com/AluVM/sonic): interface to the polynomial computer
   used in RGB.
   - `sonic-callreq`: URL-based call request API to the polynomial computer;
   - `sonic-api`: components for defining APIs for programs running the polynomial computer;
   - `hypersonic`: standard library providing runtime of the polynomial computer;
   - `sonic-persist-fs`: file-based persistence components;
   - `sonix`: command-line tool.
4. [BP-WG/bp-std](https://github.com/BP-WG/bp-std): standard Bitcoin protocol library
   for building Bitcoin wallets.
   - `bp-invoice`: Bitcoin addresses and invoices,
   - `bp-derive`: BIP-32 based extended keys and derivations,
   - `descriptors`: output script descriptos (BIP-380) implementation,
   - `psbt`: partially signed transactions implementation,
   - `bp-std`: umbrella library containing all of the above crates.
5. [RGB-WG/rgb-std](https://github.com/RGB-WG/rgb-std): standard RGB library providing
   non-consensus-level APIs and workflows.
   - `rgb-invoice`: RGB invoices,
   - `rgb-std`: standard library providing main components for RGB apps,
   - `rgb-persist-fs`: file-based persistence,
   - `rgbx`: command-line tool for working with RGB-related files.
6. [RGB-WG/rgb-interfaces](https://github.com/RGB-WG/rgb-interfaces): common data types
   used in RGB contract and interface development.

## Application layer

1. [RGB-WG/rgb](https://github.com/RGB-WG/rgb): RGB runtime and command-line for personal use
   - `rgb-descriptors`: RGB-specific wallet descriptors,
   - `rgb-psbt`: RGB-specific PSBT processing,
   - `rgb-runtime`: runtime combining wallet and RGB components,
   - `rgb-wallet`: command-line RGB wallet providing `rgb` binary.
2. [RGB-WG/rgb-node](https://github.com/RGB-WG/rgb-node): RGB Node for enterprise usage
   - `rgb-rpc`: RPC API for RGB Node,
   - `rgb-node`: RGB Node daemon, running either embedded node or standalone `rgbd` daemon;
   - `rgb-client`: RGB Node client library, also providing `rgb-cli` command-line tool.

## Toolchain

1. [strict-types/strict_types](https://github.com/strict-types/strict_types)
2. [AluVM/aluasm]
2. [UBIDECO/vesper]
3. [RGB-WG/contractum-lang]

## Organizations

The repositories listed above are structured under multiple GitHub organizations,
provided in the first part of the URL paths.

1. LNP/BP Labs
    - [LNP-BP](https://githib.com/LNP-BP): common libraries for client-side validation
    - [BP-WG](https://githib.com/BP-WG): implementation of Bitcoin protocol
    - [LNP-WG](https://githib.com/LNP-WG): implementation of Lightning protocol
2. UBIDECO Labs
    - [UBIDECO](https://githib.com/UBIDECO): common libraries for ubiquitous deterministic computing
    - [strict-types](https://githib.com/strict-types): consensus-level data encoding
    - [AluVM](https://githib.com/AluVM): 
3. RGB Consortium
    - [RGB-WG](https://githib.com/RGB-WG)

LNP/BP Labs and UBIDECO Labs are part of the same organization, a non-profit
Institute for Distributed and Cognitive Systems ([InDCS](https://indcs.org)),
based in Switzerland, Lugano and directed by [Maxim Orlovsky](https://dr.orlovsky.ch),
who has developed the original implementation of the RGB protocol.

RGB Consensus is the non-profit industrial consortium of companies contributing and
building on RGB, which is responsible for maintaining RGB protocol.

## External dependencies of the consensus layer

### Direct

Direct dependencies of `rgb-core` crate:

- `amplify`, maintained by [UBIDECO](https://githib.com/UBIDECO)
- `getrandom`

### Indirect

All dependencies of consensus-layer crates, including dependencies of dependencies,
and excluding crates already listed above.

- System-level libraries
  - `core-foundation-sys`*
  - `cpufeatures`
  - `libc`
- Derivation macros (may affect generated code)
  - `proc-macro2`
  - `syn`
  - `paste`
  - `quote`
- Cryptographic libraries
  - `block-buffer`
  - `crypto-common`
  - `generic-array`
  - `digest`
  - `ppv-lite86`
  - `rand`
  - `ripemd`
  - `secp256k1`*
  - `sha2`
  - `zerocopy`*
- Data structure libraries
  - `equivalent`
  - `hashbrown`
  - `indexmap`
  - `num-traits`
  - `thiserror`
  - `typenum`
- Time (not used in consensus)
  - `chrono`*
  - `iana-time-zone`*
- Text processing libraries (not used in consensus)
  - `ascii`
  - `base64`
  - `base85`
  - `heck`
  - `mnemonic`

\* libraries targeted to be removed from the dependencies, since their functionality is not used.

The list can be checked by running `cargo tree -e normal --prefix none | sort -u`
command inside `rgb-core` repository.
