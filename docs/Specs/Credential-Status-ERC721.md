# Introduction
The [Verifiable Credential Data Model](https://www.w3.org/TR/vc-data-model/) defines a credentialStatus field for credentials. This specification defines a credential status type called "**ERC721**", that lets a Verifiable Credential behave as a non-fungible token and as such be collected and transferred between users.

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

The token ID of a given Verifiable Credential is the SHA256 hash of the following string:

1. Define the following string and :
```json
{"id":<credential ID>,"issuer":<credential issuer>}
```
2. 