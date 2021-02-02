Related development: #21

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

## Example use cases

TODO. Artwork. supply chain tracking.

## Privacy considerations

### Personally-identifiable information
Since a hash of the whole credential is stored on the blockchain, issuers should take care of not using ERC721 tracking on credentials that contain personally-identifiable information (PII).

### TODO
Other privacy considerations?