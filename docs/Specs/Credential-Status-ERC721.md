# Introduction
The [Verifiable Credential Data Model](https://www.w3.org/TR/vc-data-model/) includes a `credentialStatus` attribute for credentials. The data model doesn't define an approach for credential statuses, but rather allows different implementations to coexist depending on the `type` attribute of the `credentialStatus` object.

This document specifies a credential status type called "**ERC721**", which allows a verifier to query on an Ethereum network the current owner of a Verifiable Credential. A verifiable credential with an `ERC721` status behaves as a non-fungible token and as such be collected and transferred between users.

# Specification

## credentialStatus

Inside the verifiable credential, the following fields are to be used inside the `credentialStatus` object:

<dl>
  <dt><code>type</code></dt>
  <dd>The status type, as defined in the Verifiable Credentials specification. The value MUST be <code>"ERC721"</code>.</dd>

  <dt><code>contractAddress</code></dt>
  <dd>A string containing the hexadecimal Ethereum address of the ERC721-compliant smart contract. For example: <code>"0x123f681646d4a755815f9cb19e1acc8565a0c2ac"</code>.</dd>

  <dt><code>networkId</code></dt>
  <dd>A string containing the hexadecimal NetworkId of the Ethereum network where the contract is deployed. In order to be able to check status, the verifier must have read access to a node running on that network.</dd>
</dl>

## Token ID

The token ID of a given Verifiable Credential is the SHA256 hash of the credential.

## Status validity

A verifier may decide to apply certain rules to consider whether an ERC721-enabled credential is valid. Some of the rules include:
- Verify that the non-fungible token is still valid inside the smart contract, i.e. that it hasn't been destroyed.
- Verify that current owner is a particular address of interest.
- Verify that current owner is **not** a particular address of interest.

Non-fungible tokens may be destroyed. A verifier may decide to consider

## Example use cases

- Check current owner or transfer ownership:
  - When buying a piece of artwork or during customs inspection
  - When renting a house or flat from its legitimate owner. Same for buying.
  - When traveling with animals.
  - As part of a pet's pedigree.
- Supply chain:
  - Passing a package from one intermediate to the next.
  - Controlling current responsibility.


## Privacy considerations

### Personally-identifiable information
Since a hash of the whole credential is stored on the blockchain, issuers should take care of not using ERC721 tracking on credentials that contain personally-identifiable information (PII).

### TODO
Other privacy considerations?