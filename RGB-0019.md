---
Author: Maxim Orlovsky
Proposed: 28 May 2025
Revision: 1; 31 May 2025
License: CC-BY
---

# Common RGB contract data types

## Motivation

Provide a set of asset-related data types which can be re-used across different RGB interfaces.

## Description

- `Amount`: a 64-bit unsigned integer;
- `AssetName`: a string, from one to 40 ASCII-printable characters;
- `Ticker`: asset ticker, which must be an alphanumberic string, starting from a letter,
  up to 8 chars, but not less than one;
- `Details`: more detailed asset description, from one to 255 ASCII-prinable characters;
- `Precision`: number of decimal digits in the fractional part in
  human-readable representation of the asset `Amount`; from zero to 18;
- `ProofOfReserves`: a pointer to a layer-1 output (like Bitcoin UTXO)
  with additional arbitrary details proving some information about reserves for a specific token.

## Specification

Details of data type definitions (using Strict types notation) which are referred from other
RGB interface standard as a types within `RGBContracts` library:

```idris
data Amount: U64

data AssetName: Std.AsciiPrintable, [Std.AsciiPrintable ^ ..0x27]

data Details: Std.AsciiPrintable, [Std.AsciiPrintable ^ ..0xfe]

data Precision: indivisible | deci | centi | milli
              | deciMilli | centiMilli | micro | deciMicro
              | centiMicro | nano | deciNano | centiNano
              | pico | deciPico | centiPico | femto
              | deciFemto | centiFemto | atto


data ProofOfReserves: utxo Bitcoin.Outpoint, proof [Byte]

data Ticker: Std.Alpha, [Std.AlphaNum ^ 1..0x7]

```