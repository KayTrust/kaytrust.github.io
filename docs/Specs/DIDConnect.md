# OpenID Connect Profile for self-sovereign identity (DIDConnect)

## Introduction

This document explains how to authenticate a user against their self-sovereign identity. This method uses [OpenID Connect (OIDC)](https://openid.net/developers/specs/), which itself builds upon the [OAuth 2.0 framework](https://tools.ietf.org/html/rfc6749).

DID Connect uses OIDC in a way that enables Self-Sovereign Identity (SSI) and favours decentralization, by combining it with DIDs and, optionally, Verifiable Credentials. That makes it different from traditional provider-centric flows in a few aspects, as detailed below.

| Aspect                   | Provider-Centric Identity          | Self-Sovereign Identity
| ------------------------- | ------------------------- | -----------------------
| Authorization Server      | Identity Provider (IdP).   | User's own identity wallet.
| Subject ID                | Determined by IdP.         | User's DID.
| Client registration flow  | Client registers against IdP (must be authorized by IdP). Client ID is an opaque string.           | No authorization needed for acting as a client. Client exposes a Verifiable Presentation, containing Verifiable Credentials issued by relevant authorities. Client ID is the URL of that presentation.
| Resource Server           | IdP's userinfo endpoint. Endpoint URL is either static or discovered through OIDC Discovery.   | Provided dynamically: endpoint URL is present in id_token ("userinfo" claim).
| Public key for id_token's signature by client | Either static or discovered through OIDC Discovery. | Provided by DID Document (DDO).
| Client's whitelisted redirect_uri endpoints | Registered and checked by the IdP. | Announced as a claim in the Client ID's Verifiable Presentation.

## Flow
The following general flow is used:

1. The clients registers (only once).
2. The client presents an authentication request in the form of a `didconnect://auth?...` URI.
3. The Authorization Server authenticates the user (for example, if the AS is a mobile app, the user should unlock the app) and obtains their consent.
4. Depending on the requested `response_type`, the Authorization Server generates a combination of identity token, grant code, and/or access token, then provides them to the client's `redirect_uri` contained in the request, according to the `response_mode` parameter (form, post or fragment).
5. The client receives the tokens and/or code.
6. If a code was received, the client gets the URL of the token endpoint from the `token_uri` parameter contained in the response and uses it to get the id_token and/or access_token.
7. If an id_token was provided, the client verifies the user's identity from it.
8. If a `userinfo` claim was present in the id_token, the client extracts the userinfo endpoint URL from it and queries it, providing the access token for authentication. The response is a verifiable presentation held by the user.

The steps are detailed in the next sections.

## Considerations about flows and Authorization Servers

OpenID Connect defines several flows: Authorization Flow, Implicit Flow and Hybrid Flow. Depending on the capacities of the client implementation and of the Authorization Server, the client's developer decides what flow should be used, and writes the request based on those capacities. That is based on the assumption that the AS is known by the developer, so that the developer can either know the AS's supported flow, or write the client app to discover them dynamically.

With the Self-Sovereign Identity model, on the other hand, the client doesn't know the AS before making the request, which makes it impossible to decide on a specific flow in the request. This is why this specification defines a way for the client to initiate a _negotiation_ about what flows it supports, so that the AS can make a decision. This is similar to Content-Type or Language negotiation in HTTP.

### 1. Client registration

1. Create a DID for your application.
2. Obtain Verifiable Credentials that come from issuers the users will trust. The credentials may be self-issued if the users trust your application's DID. The credentials should typically claim information about your DID, such as the application's name, originating organization, logo, website, etc.
3. Issue an additional Verifiable Credential containing the claim "redirect_uri" with the value of your application's whitelisted callback URL(s). That credential can be issued by your application's DID.
4. Create a Verifiable Presentation containing those credentials and held by your app's DID.
5. Place the Verifiable Presentation somewhere your users will be able to access it. The presentation's URL is your application's client ID.

### 2. Authentication request

An authentication request is a URL of the following form: `didconnect://auth?...` with the following query parameters.

| Parameter | Status | Description
|-----------|--------|------------
| `client_id` | Required | The URL of the client Verifiable Presentation.
| `redirect_uri` | Required | The callback URI the user should call after the AS generated the identity token. This URI will receive the token as a HTTP Bearer token (recommended) or as a query parameter.
| `state` | Optional | An opaque string defined by the client. That string will be provided as-is to the callback URI, as a query parameter also named `state`, allowing the client to link the response to the original request.
| `response_type` | Optional | Semicolon-separated list of flows supported by the client. Each flow is described by a combination of `id_token`, `token`, `code`, as defined by OIDC.
| `response_mode` | Optional | How the AS must issue the tokens. Must be one of `query`, `fragment`, `form_post`.

TODO: Detail the possible combinations for `response_type`.

### 3. Obtaining user consent from AS

When a user's Authorization Server receives a request from the application's client_id, it must treat `client_id` as an URL and fetch the Verifiable Presentation (VP) from it. Then, the AS must check the following conditions:

- The VP is valid and current.
- The `redirect_uri` value from the request matches one of the values present in the VP's credentials. Only valid and current credentials must be used.

Then, the AS should request user's consent after showing the user all relevant claims, based on valid and current credential from the VP. Invalid or expired credentials should be ignored.

### 4. Token generation

The AS issues `id_token` and optionally `access_token` based on the requested `response_type`. TODO: Detail.

### 5. Getting the tokens and/or code

See OpenID Connect specification. TODO: Detail.

### 6. Accessing the token (when applicable)

See OpenID Connect specification. TODO: Detail.

### 7. Verification of user's identity

The endpoint listening on `redirect_uri` receives the following query parameters (as query parameters, HTML fragment or form post depending on the requested response mode):

| Parameter | Status        | Description
|-----------|---------------|------------
| `id_token` | Required     | A JSON Web Token (JWT) containing the user's DID.
| `state`    | Optional     | The original `state` from the authentication request, if any.
| `access_token` | Optional | A Bearer token authorizing access to information about the user

The JWT contained in the `id_token` is comprised of the following fields:

| Parameter | Description
|-----------|------------
| `sub`     | The user's DID.
| `iss`| The public key of the user's device. The JWT is only valid if the device is authorised for that DID and if the JWT is correctly signed by that public key.
| `aud`| The requester's DID. The JWT is only valid if that value matches the value of `client_id` in the request.
| `state`| The state present in the request, to avoid replay attacks.
| `iat`| The date the JWT was issued at. You should check the JWT is not too old, otherwise it might be insecure.
| `exp`| The date until which the JWT is valid. You should check that the JWT is not expired, otherwise it might be insecure.
| `at_hash`| The hash of the `access_token`.
| `c_hash` | The hash of the `code`.
| `userinfo` | The URL of the Resource Server where a user's Verifiable Presentation is available.

To properly verify the user's identity, the client must:

- Verify the signature of id_token JWT against the public key present as `iss`.
- Make sure the JWT's issuer is an authorized key of the JWT's subject (i.e. the user's DID), by matching it against the DID Document.

#### Note for KayTrust
Currently, KayTrust's "GID" DID method does not support DID Documents. Until it does, the method to verify a public key is the following:

1. Compute the user's Proxy address from the DID
2. Call `proxy.owner()` to get the Identity Manager address, or use a well-known IM value for legacy DIDs.
4. Call `im.hasCap(proxy, device, "auth")` and make sure the result is `true`. To support Level of Assurance, replace "auth" with the required Level of Assurance.

### 8. Accessing the user's protected resources (when applicable)

If an access_token was provided, the client can access user's information by querying the URL referenced as `userinfo` in the id_token, and provide the access token as an `Authorization` header, as described in [OpenID Connect specification](https://openid.net/specs/openid-connect-core-1_0.html#UserInfoRequest).

The response is a Verifiable Presentation as defined by the Verifiable Credentials specification. The presentation's holder must be the user's authenticated DID. The presentation contains a `proof` attribute that allows the client to verify the holder's consent before using the included credentials.