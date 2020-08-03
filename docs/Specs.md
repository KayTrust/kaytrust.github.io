# KayTrust Specifications

## Why did you need to define anything?

_Websites_, _electronic mail_ (email), _email addresses_, _TCP connections_, _IP addresses_, _URLs_, _HTML pages_, _JPEG_. Those are known and mature concepts.

_Passwords_, _cryptographic keys_, _digital signatures_, _unique identifiers_. Also mature concepts.

_Likes_, _tweets_, _cookies_, _browser-side scripts_, user profiles. Those are a bit younger but also well known.

_Blockchain_. _Verifiable credentials_. _Digital identity_. Now we're talking about something more recent in the history of internet. Those concepts are slowly taking shape, paving their way into users' and developers' minds, and into standards organizations' discussions. RFCs are being written, W3C Working Groups are being set up, blog articles are being cited.

Since digital identity is both a young a wide domain, KayTrust's objective is to bind together the state of the art of digital identity standards and turn them into a usable package for people and for businesses.

## To the point, please.

Fair enough. So KayTrust defines the following specifications:

| Specification                                                            | Builds on top of
| ------------------------------------------------------------------------ | ----------------
| ["gid" DID method](GID-DID-Method)                                       | W3C's DID Specification
| Proxy contract ERC                                                       | Ethereum
| Identity Manager ERC                                                     | Ethereum
| [Content Attestation Registry ERC](eip-content-attestation-registry)     | Ethereum
| [Attestation Registry VC proof type](attestation-registry-proof-type)    | W3C's Verifiable Credentials Specification
| [DIDConnect OIDC Profile](DIDConnect)                                    | OpenID Connect