---
Author: Maxim Orlovsky, Kairo
Proposed: 15 May 2025
Revision: 1; 11 Jun 2025
License: CC-BY
---

# Purpose field for deterministic RGB wallets

## Background

Hierarchical deterministic key derivation, proposed in [BIP-32], allows to avoid key re-use in wallets.
[BIP-44] for the first time proposed a structure for differentiating wallets created for different
underlying blockchains and networks, to disallow re-use of keys across them.
This is achieved via introduction of so-called `cointype` field in the path derivation,
which occupies the second position (after purpose field).
[SLIP-44] standard systematizes values for `cointype` field, providing a registry of their mappings
to specific blockchains and networks.

## Specification

Under the [BIP-44] and [SLIP-44] standards the cointypes reserved for the use in RGB smart contracts
are:

|    Cointype | Network                  | Testnet     |
|------------:|--------------------------|-------------|
|   `827166'` | RGB on Bitcoin           | mainnet     |
|   `827167'` | RGB on Bitcoin or Liquid | any testnet |
|   `828942'` | RGB on Liquid            | mainnet     |

## Rationale

The base value (used in Bitcoin mainnet) is `827166`,
representing a decimal encoding of ASCII characters in "RGB" string.

The testnet value equals this value plus a value for Bitcoin (and Liquid) testnet (`1`): `827167`

The value for Liquid (`828942`) equals this base value `827166` + value used in Liquid network (`1776`).


[BIP-32]: https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki

[BIP-44]: https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki

[SLIP-44]: https://github.com/satoshilabs/slips/blob/master/slip-0044.md
