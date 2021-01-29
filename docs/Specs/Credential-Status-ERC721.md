# Introduction
The [Verifiable Credential Data Model](https://www.w3.org/TR/vc-data-model/) defines a credentialStatus field for credentials. This specification defines a credential status type called "ERC721", that lets a Verifiable Credential behave as a non-fungible token and as such be collected and transferred between users.

# Specification

The following fields are defined:

- `type`: `"ERC721"`
- `smartContract`: ERC721-compliant smart contract `"0x12345"`
- `networkId`: Ethereum network


<dl>
  <dt>type</dt>
  <dd>The status type, as defined in the Verifiable Credentials specification. The value MUST be `"ERC721"`.</dd>

  <dt>contractAddress</dt>
  <dd>A string containing the hexadecimal Ethereum address of the ERC721-compliant smart contract. For example: `"0x123f681646d4a755815f9cb19e1acc8565a0c2ac"`.</dd>

  <dt>networkId</dt>
  <dd>A string containing the hexadecimal NetworkId of the Ethereum network where the contract is deployed. In order to be able to verify the credential, the verifier must have read access to a node running on that network.</dd>
</dl>

