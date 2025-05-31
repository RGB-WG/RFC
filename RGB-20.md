---
Author: Maxim Orlovsky
Proposed: 27 May 2025
Revision: 1; 27 May 2025
---

# Standard for fungible assets

## Motivation

To have a standard for similar to ERC-20 and other fungible token standards.

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

RGB-20 provides several important additional capabilities, usually absent in other blockchain-based token standards.
We list these capabilities in the subsections below.

### Renaming

The metadata of the asset may be changed by an issuer.

The issuers should decide at the contract creation whether they opt in for a such possibility.

### Adjustable precision

The issuer may change the precision of an asset. The common use case for that is an operation,
similar to the stock split, where each owner of the assets gets their asset multiplied on a certain
coefficient, proportional to 10. This is useful in cases when the asset price is high, and it is
desirable to make the full asset more accessible.

In reality, the real number of assets is measured in the smallest atomic integer units, which do never change.
However, in terms of user interface, they are shown as a certain fractional,
which is the real number of asset atoms, divided by 10 powered to the number of asset decimals.
This is how the precision works: it defines that number of the decimals.

The only thing the issuer can change is how the number of asset atoms is represented in UI.
Such changes are also only possible, when the issuer opted in for them at the moment of the contract
issue; and the issuer has always a right to opt out of it later.
All these changes are detectable and verifiable by the contract owners.

### Inflation with a restricted cap

RGB-20 assets may be inflated, or may be prohibited from being inflated.
The standard defines the precise form a user may detect the ability of the issuer to inflate the asset,
as well as to restrict the maximum number of secondary issues by a certain cap.

### Provable burns

An issuer may provably burn assets, notifying the marked about the reduced supply.

### Proof of reserves

Another distinguishing feature of the standard is an ability to reference some existing Bitcoin
UTXO as a token-specific "proof of reserves." A wallet may check that proof and make sure that if
the token features such a proof of reserves, that it is indeed present and contains a promised value.

## Specification

### Published state

| Name      | Required | Is multiple | Data type              | Description                                                                  |
|-----------|----------|-------------|------------------------|------------------------------------------------------------------------------|
| name      | yes      | can be      | String                 | History of asset name                                                        |
| ticker    | yes      | can be      | String                 | History of asset ticker                                                      |
| terms     | no       | can be      | String                 | History of asset terms and conditions                                        |
| precision | yes      | can be      | integer (from 0 to 18) | History of asset precision (number of decimals)                              |
| issued    | yes      | can be      | U64, ProofOfReserves   | Number of assets issued at primary (genesis) or secondary (`inflate`) issues |
| burned    | no       | can be      | U64                    | Number of assets burned with an `inflate` operation                          |

Legend: comma in "Data types" separates verifiable part of the global state from unverifiable and incompressible data.

### Computed state

| Name      | Required | Data type             | Description                                                                                                                      |
|-----------|----------|-----------------------|----------------------------------------------------------------------------------------------------------------------------------|
| name      | yes      | String                | Actual asset name                                                                                                                |
| ticker    | yes      | String                | Actual asset ticker                                                                                                              |
| terms     | no       | String                | Asset terms and conditions                                                                                                       |
| precision | yes      | integer, from 0 to 18 | Actual asset precision (number of decimals)                                                                                      |
| supply    | yes      | U64                   | Circulating asset supply (see [Get circulating asset suppu](#get-circulating-asset-supply) section for the details)              |
| maxSupply | no       | U64                   | Maximum supply of the asset (ss [Get maximum possible asset supply](#get-maximum-possible-asset-supply) section for the details) |

### Owned state

| Name           | Is multiple | Data type    | Description                                               |
|----------------|-------------|--------------|-----------------------------------------------------------|
| balance        | yes         | U64          | Balance of the assets assigned to a single-use seal owner |
| inflationRight | no          | Optional U64 | Size of allowed inflation assigned to secondary issuer(s) |
| controlRight   | no          | -            | The right to perform change in the asset details          |

### Operations

| Name     | Required? | Publishes                          | Spends         | Assigns         | Description                                     |
|----------|-----------|------------------------------------|----------------|-----------------|-------------------------------------------------|
| transfer | yes       | -                                  | balance+       | balance*        | Transfer assets                                 |
| inflate  | no        | issued                             | inflationRight | inflationRight? | Secondary inflation, asset burn or re-issue     |
| change   | no        | ticker?, name?, precision?, terms? | controlRight   | controlRight?   | Changing asset name, ticker, terms or precision |

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

### Get the history of inflation

Take the values in `global.issued` and `global.burned`, which are arrays.
The individual state entries in that array contain the information of the height at which the issue
or burn has happened and the number of the issued/burned assets at that point.

### Get circulating asset supply

For a non-inflatable asset, `computed.supply` must always be defined and contain circulating
supply which is always equal to the total supply.

For an inflatable asset, check `computed.supply`, which reflects the known part of the issued supply
minus burned supply (if any). However, there might be issues or burns that are not known.
This can be detected by checking whether seal definition in `owned.inflationRight` is spent by some
bitcoin transaction: if it is spent, the total circulating supply is not known and the wallet should
request the latest updates from the asset issuer.

### Get maximum possible asset supply

Take `computed.maxSupply` property.
If it is not set or absent, the maximum possible supply is unlimited.

### Get the latest asset details

Use `computed.name`, `computed.ticker`, `computed.terms` and `computed.precision`
for the latest known asset information.

Check `global.name`, `global.ticker`, `computed.terms` and `global.precision`
which contain the whole known history of asset information changes.

Check `owned.controlRight` seal definition; if it belongs to an unspent UTXO, you are in possession
of the most recent asset information. If not, contact the contract issuer for an update of the
contract consignment.

### How to change the asset details

To rename an asset, one needs to be in control of `controlRight` assignment. 
If you have the right, use `change` call.

### How to use proof of reserves
