# OpenID Connect Profile for self-sovereign identity (DIDConnect)

## Introduction

This document explains how to authenticate a user against their self-sovereign identity. This method is inspired by [OpenID Connect (OIDC)](https://openid.net/developers/specs/), which itself builds upon the [OAuth 2.0 framework](https://tools.ietf.org/html/rfc6749).

OAuth 2.0 defines the following actors:

| OAuth 2.0                 | OpenID Connect       | DID Connect
| ------------------------- | -------------------- | -----------------------
| Resource Owner (RO)       | User                 | User's identity (DID)
| Authorization Server (AS) | Authorization Server | User's controlling device
| Client                    | Relying Party (RP)   | Relying Party (RP)
| Resource Server (RS)      | Userinfo endpoint    | Online location where a user's presentation is available


The following points differ between provider-centric and self-sovereign identity:

| Concept                   | Provider-centric          | Self-sovereign
| ------------------------- | ------------------------- | -----------------------
| Authorization Server      | Identity Provider (IdP)   | User's own identity wallet
| Subject ID                | Determined by IdP         | User's DID
| Client registration flow  | Client registers against IdP (must be authorized by IdP). Client ID is an opaque string.           | No authorization needed for acting as a client. Client exposes a Verifiable Presentation, containing Verifiable Credentials issued by relevant authorities. Client ID is the URL of the presentation.
| Resource Server           | IdP's userinfo endpoint. Endpoint URL is either static or discovered through OIDC Discovery   | Determined by the user. Endpoint URL is present in id_token
| Public key for id_token's signature by client | Either static or discovered through OIDC Discovery | Provided by DID Document (DDO) 

## Flow
When the Authorization Server is a mobile app (e.g. KayTrust Wallet running on a trusted device), OAuth 2.0 Implicit Flow is used:

1. The client requests authentication from a user by presenting them an authentication request in the form of a URI.
2. The user signs into their Authorization Server.
3. The AS generates an identity token based on the scope defined in the request, then redirects the user agent to the client's `redirect_uri` contained in the request.
4. The client receives the identity token and validates it.

### Registering the client application (only done once)

Before users can trust a client's redirect endpoint, the client needs to publish it as a credential. This is similar to registering an application with a centralized OAuth provider, except that the client can do it independently by self-issuing a credential containing a `redirect_uri` claim with all whitelisted URIs. How users access such credential is out of the scope of this document.

### Requesting authentication

An authentication request is a URL of the following form: `didconnect://share?client_id={DID}&redirect_uri={URI}&state={STATE}`

The following parameters are mandatory:

- `client_id`: The DID of the client application.
- `redirect_uri`: The callback URI the user should call after the AS generated the identity token. This URI will receive the token as a HTTP Bearer token (recommended) or as a query parameter.
- `state`: An opaque string defined by the RP. That string will be provided as-is to the callback URI, as a query parameter also named `state`, allowing the RP to link the response to the original request.

The following parameter is optional:

- `description`: A free-form text that explains to the user why authentication is being requested.

### Verifying the user's identity

The service running at `redirect_uri` receives a query with the following query parameters:

- `id_token`: a JSON Web Token (JWT) containing proof of the user's DID
- `state`: the original `state` presented in the authentication request
- `access_token` (optional): a Bearer token authorizing access to information about the user

The JWT contained in the `id_token` is comprised of the following fields:

- `sub`: The user's DID.
- `iss`: The public key of the user's device. The JWT is only valid if the device is authorised for that DID and if the JWT is correctly signed by that public key.
- `aud`: The requester's DID. The JWT is only valid if that value matches the value of `client_id` in the request.
- `state`: The state present in the request, to avoid replay attacks.
- `iat`: The date the JWT was issued at. You should check the JWT is not too old, otherwise it might be insecure.
- `exp`: The date until which the JWT is valid. You should check that the JWT is not expired, otherwise it might be insecure.
- If `access_token` was provided:
    - `at_hash`: The hash of the `access_token`.
    - `userinfo`: The URL of the Resource Server where a user's Verifiable Presentation is available.

To properly verify the user's identity, the client must:

- Verify the signature of id_token JWT against the public key present as `iss`.
- Make sure the JWT's issuer is an authorized key of the JWT's subject (i.e. the user's DID), by matching it against the DID Document.

#### Note for KayTrust
Currently, KayTrust "GID" DID method does not support DID Documents. Until it does, the method to verify a public key is the following:

1. Compute the user's Proxy address from the DID
2. Call `proxy.owner()` to get the Identity Manager address, or use a well-known IM value for legacy DIDs.
4. Call `im.hasCap(proxy, device, "auth")` and make sure the result is `true`. To support Level of Assurance, replace "auth" with the required Level of Assurance.

### Optional: Accessing the user's protected resources
If an access_token was provided, the client can access user's information by querying the URL referenced as `userinfo` in the id_token, and provide the access token as an `Authorization` header, as described in [OpenID Connect specification](https://openid.net/specs/openid-connect-core-1_0.html#UserInfoRequest).

The response is a Verifiable Presentation as defined by the Verifiable Credentials specification. The presentation contains a `proof` attribute that allows the client to verify the holder's consent before using the credentials contained in the presentation.
