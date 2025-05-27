---
Author: Maxim Orlovsky
Proposed: 27 May 2025
Revision: 1; 27 May 2025
---

# Standard for fungible assets

## Motivation

To have a standard for RGB working in a manner similar to ERC-20 and other fungible token standards.

## Description

RGB-20 standard offers fungible assets (or tokens), which supply is split between individual owners
and can be transferred between them in a permissionless manner, without acknowledgment of an issuer.

Features of RGB-20 assets:
- inflation, which may be restricted in cap;
- asset burning;
- ability to change the asset nominal (`precision`), which is similar to the stock split procedure;
- ability to change the asset name, ticker and terms & conditions.

At the same time, it is possible to have assets which are provably
- non-inflatable;
- can't be renamed to can't change terms & conditions.

## Specification

### Published state

| Name      | Required | Is multiple | Data type             | Description                                            |
|-----------|----------|-------------|-----------------------|--------------------------------------------------------|
| name      | yes      | can be      | String                | History of asset names                                 |
| ticker    | yes      | can be      | String                | History of asset tickers                               |
| terms     | no       | can be      | String                | Asset terms and conditions                             |
| precision | yes      | can be      | integer, from 0 to 18 | History of asset precision - number of decimals        |
| issued    | yes      | can be      | U64                   | Number of assets issued at primary or secondary issues |

### Computed state

| Name         | Required | Data type             | Description                                                                                                                         |
|--------------|----------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| name         | yes      | String                | History of asset names                                                                                                              |
| ticker       | yes      | String                | History of asset tickers                                                                                                            |
| terms        | no       | String                | Asset terms and conditions                                                                                                          |
| precision    | yes      | integer, from 0 to 18 | History of asset precision - number of decimals                                                                                     |
| issuedSupply | yes      | U64                   | The known total issued asset supply (see [Get circulating asset suppu](#get-circulating-asset-supply) section for the details)      |
| maxSupply    | no       | U64                   | The maximum supply of an asset (ss [Get maximum possible asset supply](#get-maximum-possible-asset-supply) section for the details) |

### Owned state

| Name           | Is multiple | Data type    | Description                                               |
|----------------|-------------|--------------|-----------------------------------------------------------|
| balance        | yes         | U64          | Balance of the assets assigned to a single-use seal owner |
| inflationRight | no          | Optional U64 | Size of allowed inflation assigned to secondary issuer(s) |
| controlRight   | no          | -            | The right to perform renomination                         |

### Operations

| Name       | Required? | Publishes                       | Spends         | Assigns         | Description                                 |
|------------|-----------|---------------------------------|----------------|-----------------|---------------------------------------------|
| transfer   | yes       | -                               | balance+       | balance*        | Transfer assets                             |
| inflate    | no        | issued                          | inflationRight | inflationRight? | Secondary inflation, asset burn or re-issue |
| renominate | no        | ticker, name, precision, terms? | controlRight   | controlRight?   | Changing asset name, ticker and precision   |

Legend:
- `?` means that the argument of the call may be absent;
- `*` means that the argument of the call may be absent or contain multiple elements;
- `+` means that the argument of the call may contain multiple elements,
  but at least one element must be present.

## Guidelines

Here you will find guidelines to how software should handle the most common tasks over the contract
state.

### Compute total user balance

Iterate over `owned.balance`, and filter the balances which belong to the UTXOs,
defined in the current wallet. Sum up that balance to get the total user funds.

### Detect whether an asset can be further inflated

Check for `owned.inflation` right, which, if present, defines an ability for the asset inflation.
If it has a value, this value provides a limit for the further asset inflation.
If the value is not present, the possible inflation is unlimited.

### Get the history of secondary issues (inflation)

Take the value in `global.issued`, which is an array.
The individual state entries in that array contain the information of the height at which the issue
has happened and the number of the issued assets at that point.

### Get circulating asset supply

For a non-inflatable asset, `computed.issuedSupply` must always be defined and contain
the total supply.

For an inflatable asset, check `computed.issuedSupply`, which reflects the known part of the
issued supply. However, there might be issues that are not known. This can be detected by
checking whether seal definition in `owned.inflationRight` is spent by some bitcoin transaction:
if it is spent, the total circulating supply is not known and the wallet should request the
latest updates from the asset issuer.

### Get maximum possible asset supply

Take `computed.maxSupply` property.
If it is not set or absent, the maximum possible supply is unlimited.

### Get tasset precision and supply

Use `computed.name`, `computed.ticker` and `computed.precision` for the latest known asset
information.

Check `global.name`, `global.ticker` and `global.precision` which contain the whole known history
of asset information changes.

Check `owned.controlRight` seal definition; if it belongs to an unspent UTXO, you are in possession
of the most recent asset information. If not, contact the contract issuer for an update of the
contract consignment.

### How to rename an asset

To rename an asset, one needs to be in control of `controlRight` assignment. 
If you have the right, use `renominate` call.
