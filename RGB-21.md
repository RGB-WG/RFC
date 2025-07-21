---
Author: Maxim Orlovsky
Proposed: 28 May 2025
Revision: 1; 28 May 2025
License: CC-BY
---

# Standard for non-fungible assets

## Motivation

To have a standard for RGB similar to ERC-271 and ERC-404,
as well as other non-fungible token standards.

## Description

RGB-21 standard offers non-fungible assets (or tokens) which may be separately owned
and can be transferred between the owners in a permissionless manner,
without acknowledgment of an issuer.

Features of RGB-21 NFTs:
- allows single NFT and collection of NFTs;
- NFTs (both single and in a collection) can be owned in pre-defined fractions;
- collections may be (or may be not) extendable with new tokens;
- ability to change the asset name, ticker and terms & conditions;
- ability to add engravings (additional data, like images) on top of NTFs.

RGB-21 provides several important additional capabilities,
usually absent in other blockchain-based token standards.
We list these capabilities in the subsections below.

### Differentiated fractionality

First, one may have fractional and non-fractional tokens under the same contract.
The token fractionality may be also seen as a "number of token copies," which is common in the
art world; and in this case that number defines the token "rarity."
Different tokens may have different "rarity."

### Multiple token transfers

A single transfer may move multiple tokens and fractions of tokens.

### Single tokens vs. collections, which can be extended

The interface offers ability to create both a singular NFT or a collection of NFTs.
The collections may be fixed in size, - or extendable later with more tokens.

### Fixed and extendable collections

The issuer decides on whether a token or set of tokens can be extended in the future.
Thus, users are always provided with the information about the total maximum issue of the NFTs.

One of the options for the issuer is to postpone the issue of the token collection,
first creating a contract with zero tokens and then adding them over time.

### Renaming collection

The metadata of the collection may be changed by an issuer.

The issuers should decide at the contract creation whether they opt in for a such possibility.

At the same time, the metadata of individual tokens, once they are issued, can't be changed.

### Client-side data

Unlike in all other systems, the client-side model of RGB smart contracts allow to put even
voluminous NFT data right into the contract itself, not paying any gas fees etc.

Thus, for the first time, RGB unleashes a way for truly decentralized NFTs, not requiring any
centralized server or Internet access.

At the same time, for the sake of updates, a creator of NFT may still use a server-based content,
if needed.

### Engravings

One of the distinguishing features, which is possible with RGB and its client-side hosted data model,
is the ability for the token owners to attach arbitrary additional data on top of the token itself.
These are so-called _engravings_.

One may think of engravings like of "autographs" on top of books, made by authors, or on top of
sport balls, made my sport players. These engravings helps to further differentiate individual
tokens and increase their value.

The engraving feature has to be enabled by the NFT creator.

### Proof of reserves

Another distinguishing feature of the standard is an ability to reference some existing Bitcoin
UTXO as a token-specific "proof of reserves." A wallet may check that proof and make sure that if
the token features such a proof of reserves, that it is indeed present and contains a promised value.

## Specification

### Custom data types

- `TokenNo`: a 32-bit unsigned integer;
- `TokenFractions`: a 64-bit unsigned integer;
- `Nft`: a structure, which combines information about the token number with the number of known token fractions;
- `NftSpec`: detailed token specification, which includes names and related media data (see below);
- `OwnedNft`: similar structure, but used in the context of token ownership
  (i.e., fractions means not total fractions, but a number of fractions allocated to a specific output);
- `EmbeddedMedia`: an embedded media data (se below).

Details of data type definitions (using Strict types notation):

```idris
data Attachment: mime MediaType, digest [Byte ^ 32]

data EmbeddedMedia: mime MediaType, data [Byte]

data MediaRegName: Std.AlphaSmall, [MimeChar ^ ..0x3f]

data MediaType: type MediaRegName, subtype MediaRegName?, charset MediaRegName?

data MimeChar: excl#33 | hash#35 | dollar | amp#38
             | plus#43 | dash#45 | dot | zero#48
             | one | two | three | four
             | five | six | seven | eight
             | nine | caret#94 | lodash | a#97
             | b | c | d | e
             | f | g | h | i
             | j | k | l | m
             | n | o | p | q
             | r | s | t | u
             | v | w | x | y
             | z


data Nft: tokenNo TokenNo, fractions TokenFractions

data NftEngraving: appliedTo TokenNo, content EmbeddedMedia

data NftSpec: name RGBContract.AssetName?,
              embedded EmbeddedMedia,
              external Attachment?,
              reserves RGBContract.ProofOfReserves?

data TokenFractions: U64

data TokenNo: U32
```

### Published state

| Name         | Required | Is multiple | Data type              | Description                                |
|--------------|----------|-------------|------------------------|--------------------------------------------|
| name         | yes      | can be      | String                 | History of asset name                      |
| ticker       | yes      | can be      | String                 | History of asset ticker                    |
| terms        | no       | can be      | String                 | History of asset terms and conditions      |
| maxTokens    | no       | no          | U32                    | Maximal number of tokens                   |
| maxFractions | no       | no          | TokenFractions         | Maximal number of fractions in any token   |
| baseUrl      | no       | no          | String                 | History of the base URL for external media |
| token        | no       | can be      | Nft, NftSpec           | Individual tokens in the collection        |
| engraving    | no       | yes         | TokenNo, EmbeddedMedia | An engraving on top of a token             |

Legend: comma in "Data types" separates verifiable part of the global state from unverifiable and incompressible data.

### Computed state

| Name       | Required | Data type                  | Description                                             |
|------------|----------|----------------------------|---------------------------------------------------------|
| name       | yes      | String                     | Actual asset name                                       |
| ticker     | yes      | String                     | Actual asset ticker                                     |
| terms      | no       | String                     | Asset terms and conditions                              |
| baseUrl    | no       | String                     | An actual string for the base URL for external media    |
| count      | yes      | U32                        | The number of already issued tokens                     |
| tokens     | yes      | Nft -> NftSpec             | Individual tokens of the collection                     |
| engravings | no       | TokenNo -> [EmbeddedMedia] | A map from a token id to an array of applied engravings |

### Owned state

| Name           | Is multiple | Data type       | Description                                               |
|----------------|-------------|-----------------|-----------------------------------------------------------|
| balance        | yes         | OwnedNft        | Information about the owned token                         |
| inflationRight | no          | TokenFractions? | Size of allowed inflation assigned to secondary issuer(s) |
| controlRight   | no          | -               | The right to perform change in the asset details          |

### Operations

| Name     | Required? | Reads     | Publishes                        | Spends         | Assigns         | Description                                         |
|----------|-----------|-----------|----------------------------------|----------------|-----------------|-----------------------------------------------------|
| transfer | yes       |           | -                                | balance+       | balance*        | Transfer assets                                     |
| engrave  | no        |           | engravings                       | balance        | balance?        | Transfer a single token and add an engraving on top |
| inflate  | no        | maxTokens | token*                           | inflationRight | inflationRight? | Add tokens to the collection                        |
| change   | no        |           | ticker?, name?, terms?, baseUrl? | controlRight   | controlRight?   | Changing asset name, ticker, terms or baseUrl       |

Legend:
- `?` means that the argument of the call may be absent;
- `*` means that the argument of the call may be absent or contain multiple elements;
- `+` means that the argument of the call may contain multiple elements,
  but at least one element must be present;
- `[..]`, a value in square brackets, indicate an array;
- `{..}`, a value in square brackets, indicate an ordered set, where all elements are unique;
- `.. -> ..`, indicates a map type, from keys to values.

## Guidelines

Here you will find guidelines to how software should handle the most common tasks over the contract
state.

### Get a list of tokens for a user

Iterate over `owned.balance`, and filter the entries which belong to the UTXOs,defined in the current wallet.
Sum up that balance for each token no to get the total fractions owned by the user.

### Detect whether new tokens can be added to the collection

Check for `owned.inflation` right, which, if present,
defines a possibility for the collection to be extended with additional NFTs.

You also need to check whether `computed.maxTokens` is equal to the `computed.count`.
If the `computed.maxTokens` is absent, this means the new tokens can be added to the collection
by the issuer, unlimitedly.

### Get maximum possible asset supply

Take `computed.maxTokens` property.
If it is not set or absent, the maximum possible supply is unlimited.

### List all NFTs

Take the values in `computed.tokens`, which is an map of all token definitions, from token numbers,
to token specifications.

NB: Do not use `global.token`, since it is an internal data structure, which may contain repeated
objects, filtered out in `computed.token`.

### Get circulating asset supply

For a non-inflatable asset, `computed.supply` must always be defined and contain circulating
supply which is always equal to the total supply.

For an inflatable asset, check `computed.supply`, which reflects the known part of the issued supply
minus burned supply (if any). However, there might be issues or burns that are not known.
This can be detected by checking whether seal definition in `owned.inflationRight` is spent by some
bitcoin transaction: if it is spent, the total circulating supply is not known and the wallet should
request the latest updates from the asset issuer.

### Engravings

To get all engravings applied to some specific token, use `computed.engravings` state field,
which provides a map from a token number to the array of all related engravings, in the order
of their application.

### Get the latest asset details

Use `computed.name`, `computed.ticker` and `computed.baseUrl`
for the latest known asset information.

Check `global.name`, `global.ticker` and `global.baseUrl`
which contain the whole known history of asset information changes.

Check `owned.controlRight` seal definition; if it belongs to an unspent UTXO, you are in possession
of the most recent asset information. If not, contact the contract issuer for an update of the
contract consignment.

The contract terms are immutable, and if defined, are always present in the singular `globa.terms`
value.

### Get associated token media

Each token can embed media right into its specification. However, some issuers may want to host
the tokens on their website. This can be done by using `external` field in the `NftSpec` structure.

The users should check this field, and, if present,
append its `digest` value after `computed.baseUrl` to get the full HTTP(s) address for the token media.

### How to change the asset details

To rename an asset, one needs to be in control of `controlRight` assignment.
If you have the right, use `change` call.
