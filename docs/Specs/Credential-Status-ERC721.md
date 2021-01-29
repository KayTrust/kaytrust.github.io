# Introduction
The [Verifiable Credential Data Model](https://www.w3.org/TR/vc-data-model/) defines a credentialStatus field for credentials. This specification defines a credential status type called "ERC721", that lets a Verifiable Credential behave as a non-fungible token and as such be collected and transferred between users.

# Definition

The following fields are defined:

- `type`: `"ERC721"`
- `smartContract`: ERC721-compliant smart contract `"0x12345"`
- `networkId`: Ethereum network