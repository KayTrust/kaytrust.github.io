# Ethereum Attestation Registry Proof Type

## Abstract

This document introduces a new embedded proof type for Verifiable Credentials and Verifiable Presentations, as described in [Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/)'s ["Proofs" section](https://w3c.github.io/vc-data-model/#proofs-signatures), currently published by the W3C Credentials Community Group.

## Status of this Document
This specification is an unofficial draft. It is provided as a reference for people and organisations who wish to implement Ethereum-based proofs.

## Introduction

The Verifiable Credentials specification [describes](https://w3c.github.io/vc-data-model/#proofs-signatures) a modular mechanism for declaring how a given credential must be verified.

This specification introduces a new type of proof called "EthereumAttestationRegistry2019". This type of proof is based on a trusted smart contract deployed on an Ethereum blockchain accessible by the verifier. Verification of a credential containing this type of proof is done by calls a specific function to the smart contract.

This document extends the [Verifiable Credentials specification](https://w3c.github.io/vc-data-model/) and assumes that the terminology and concepts of that specification are known.

## Specification

An "Ethereum attestation registry (2019)" proof is represented by an object contained in the `"proof"` section of a Verifiable Credential. As such, it is considered an "embedded proof".

It must contain the following attributes:

<dl>
  <dt>type</dt>
  <dd>The proof type, as defined in the Verifiable Credentials specification. The value MUST be `"EthereumAttestationRegistry2019"`.</dd>

  <dt>contractAddress</dt>
  <dd>A string containing the hexadecimal Ethereum address of the smart contract that allows the verification of the credential. For example: `"0x123f681646d4a755815f9cb19e1acc8565a0c2ac"`. That contract must implement **EIP-XXX (to be defined): Content Attestation Registry**.</dd>

  <dt>networkId</dt>
  <dd>A string containing the hexadecimal NetworkId of the Ethereum network where the contract is deployed. In order to be able to verify the credential, the verifier must have read access to a node running on that network.</dd>
</dl>

For a proof to be valid, it MUST be possible to determine an Ethereum address from the `"issuer"` attribute. For example, the `"ev"` DID method allows to determine the Ethereum address from a DID.

## Example

The following example shows a Verifiable Credential with an Ethereum Attestation Registry proof.

```json
{
  "@type": "VerifiableCredential",
  "@context": "https://schema.org",
  "credentialSubject": {
    "@id": "did:example:abcd",
    "name": "John Doe",
    "birthdate": "2018-01-01"
  },
  "issuer": "did:ev:2uukHPBYMjdZPkg4p5ZjipKHzkaXLr4T5ut",
  "proof": {
      "type": "EthereumAttestationRegistry2019",
      "contractAddress": "0x123f681646d4a755815f9cb19e1acc8565a0c2ac",
      "networkId": "0x19"
    }
}
```

## Proof Generation Method

The following steps MUST be applied by a credential's issuer in order to generate an Ethereum Attestation Registry proof of the credential:

1. Create the proof object as defined in previous sections and add it to the credential as a `proof` attribute. If `proof` already exists, ensure it becomes an array containing previous values + the new object.
2. At this point, the issuer MAY distribute the credential. However, it won't be possible to verify the proof until the rest of the steps are applied, so it is RECOMMENDED to wait until the end of the proof generation to distribute the credential.
3. Compute the credential's hash:
    1. Temporarily strip the whole `"proof"` attribute from the credential, even if it contains multiple proofs.
    2. Serialize the resulting object as a string. TODO: Describe the serialization method (it must be deterministic).
    3. Compute the SHA256 hash of the string.
4. Determine the Ethereum address corresponding to the issuer's URI.
5. Decide on the validity timerange of the attestation in the form of a integer couple `(iat, exp)`, where `iat` is the start date and `exp` is the expiration date.
6. Decide on the Ethereum network and smart contract to be used for storing the attestation.
7. Send a transaction to the Ethereum network and smart contract containing a call to the `attest(bytes32 hash, uint iat, uint exp)` function, where `hash`, `iat` and `exp` are the values computed above.
9. Keep and distribute the resulting credential.

## Proof Revocation Method

This proof type allows for an already-emitted credential to be revoked by its original issuer. The following steps MUST be applied by the credential's issuer in order to revoke an Ethereum Attestation Registry proof:

1. Compute the credential's hash:
    1. Temporarily strip the whole `"proof"` attribute from the credential, even if it contains multiple proofs.
    2. Serialize the resulting object as a string. TODO: Describe the serialization method (it must be deterministic).
    3. Compute the SHA256 hash of the string.
2. Determine the Ethereum address corresponding to the issuer's URI.
3. Send a transaction to the Ethereum network and smart contract listed in the proof to be revoked, containing a call to the `attest(bytes32 hash, 0, 0)` function, where `hash` is the credential's hash.

Note that no further action is required, since the attestation registry will be updated as soon as the transaction goes through. The attestation registry will also notify interested parties through an Ethereum event that the proof is no longer valid.

## Proof Verification Method

The following steps MUST be applied by a credential's verifier in order to verify an Ethereum Attestation Registry proof of the credential:

1. Compute the credential's hash:
    1. Temporarily strip the whole `"proof"` attribute from the credential, even if it contains multiple proofs.
    2. Serialize the resulting object as a string. TODO: Describe the serialization method (it must be deterministic).
    3. Compute the SHA256 hash of the string.
2. Determine the Ethereum address corresponding to the issuer's URI.
3. Call the `attestations(hash, issuer)` function of the smart contract listed in the proof, where `hash` is the credential's hash and `issuer` is issuer's Ethereum address and store the returned value couple as `(iat, exp)`. Those are the start date and end date, respectively, specified by the issuer when signing the credential.
4. Verify that the time range is acceptable. The following two conditions below SHOULD both be verified for the credential to be considered valid. However, the verifier MAY apply a different policy if the use case justifies it.
    - `iat` is not `0` and is lower than the time of the verification.
    - `exp` is `0` or is higher than the time of the verification.

## Performance Considerations

With this proof type, an Ethereum transaction must be executed for each new attestation. This specification should be improved in the future to allow more scalable approaches.