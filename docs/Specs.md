# KayTrust Specifications

## DID methods

KayTrust uses the standard DID protocol for identifiers, and defines a ["ev" DID method](https://github.com/KayTrust/did-method-ev) based on [smart contracts](/Specs/Proxy-Contract-ERC).

More DID methods might be defined in the future.

| Specification                                  | Builds on top of        | What is it good for?
| ---------------------------------------------- | ----------------------- | --------------------
| ["ev" DID method](https://github.com/KayTrust/did-method-ev)      | W3C's DID Specification | Ethereum-based DIDs
| [Proxy contract ERC](/Specs/Proxy-Contract-ERC)| Ethereum                | Transaction forwarding, on-chain representation, single Ethereum addresses
| Identity Manager ERC                           | Ethereum                | Flexible controlling logic for Proxy contracts

## Verifiable credentials and Presentations

Besides identifiers, the point of an identity is to have credentials associated to it. A credential answers the question *"Who are you?"* and contains one or more key-value claims (e.g. birth date, name, qualifications, citizenships, etc.) about an entity called subject, issued by another entity called issuer. The Verifiable Credentials Working Group at the W3C is defining a standard that KayTrust follows.

Both Verifiable Credentials (VC) and Verifiable Presentations (VP) contain proofs, which is what makes them verifiable. The VC specification doesn't enforce a specific proof algorithm but describes the articulation between a credential/presentation and a specific proof method. Implementers are free to come up with their own proof method or to follow someone else's.

The [draft ERC](/Specs/Content-Attestation-Registry-ERC) (Ethereum Request for Comments) describes a way for any entity to attest arbitrary content on a smart contract. There is a corresponding [proof type](/Specs/Ethereum-Attestation-Registry-Proof-Type) that enables to use that attestation registry inside a Verifiable Credential or a Verifiable Presentation.

| Specification                                                         | Builds on top of        | What is it good for?
| --------------------------------------------------------------------- | ----------------------- | --------------------
| [Content Attestation Registry ERC](/Specs/Content-Attestation-Registry-ERC)  | Ethereum                | Attesting any kind of content on-chain
| [Attestation Registry VC proof type](/Specs/Ethereum-Attestation-Registry-Proof-Type) | W3C's Verifiable Credentials data model | Using a Content Attestation Registry as proof of a VC or a VP

## Real-world, self-sovereign authentication: "DID Connect"

KayTrust introduces a way for identity owners (a.k.a. subjects) to authenticate on third-party apps. We propose using OpenID Connect, only in a self-sovereign fashion. The trick is to use as Authorization Server the identity owner's own device, as opposed to a predefined AS in traditional services.

| Specification                         | Builds on top of | What is it good for?
| ------------------------------------- | ---------------- | ------------------------------------
| [DIDConnect OIDC Profile](https://github.com/KayTrust/did-connect) | OpenID Connect   | Self-sovereign use of OpenID Connect

## Schemas

KayTrust mostly relies on well-known schemas, such as the great work done by the schema.org community. However, when the need arises, additional schemas are defined.

| Schema                                            | Purpose
| ------------------------------------------------- | --------------------------------------------------
| [Trusted Credentials](https://github.com/KayTrust/schemas/blob/draft/Trusted-Credentials.md) | Chain of Trust for Verifiable Credentials


## Why did you need to define anything?

_Websites_, _electronic mail_ (email), _email addresses_, _TCP connections_, _IP addresses_, _URLs_, _HTML pages_, _JPEG_. Those are known and mature concepts.

_Passwords_, _cryptographic keys_, _digital signatures_, _unique identifiers_. Also mature concepts.

_Likes_, _tweets_, _cookies_, _browser-side scripts_, user profiles. Those are a bit younger but also well known.

_Blockchain_. _Verifiable credentials_. _Digital identity_. Now we're talking about something more recent in the history of internet. Those concepts are slowly taking shape, paving their way into users' and developers' minds, and into standards organizations' discussions. RFCs are being written, W3C Working Groups are being set up, blog articles are being cited.

Since digital identity is both a young a wide domain, KayTrust's objective is to bind together the state of the art of digital identity standards and turn them into a usable package for people and for businesses.