# OpenID Connect Profile for self-sovereign identity (DIDConnect)

## Introduction

This document describes a method, nicknamed "DID Connect", to authenticate a user against their [Decentralized Identifier (DID)](https://www.w3.org/TR/did-core/). This method uses [OpenID Connect (OIDC)](https://openid.net/developers/specs/), which itself builds upon the [OAuth 2.0 framework](https://tools.ietf.org/html/rfc6749).

The objective of DID Connect is to extend OIDC in a way that makes it as resource-owner-centric as possible, to enable Self-Sovereign Identity (SSI) scenarios. That decentralization makes DID Connect different from traditional provider-centric OIDC flows in a few aspects, as detailed below.

| Aspect                    | Provider-Centric Identity          | Self-Sovereign Identity
| ------------------------- | ------------------------- | -----------------------
| Subject ID (`sub`)               | IdP-assigned.         | User's DID.
| Client registration flow  | Client registers against IdP (must be authorized by IdP).           | No authorization needed by a central entity for acting as a client. Authorization is whatever set of Verifiable Credentials the client is able to present.
| Client information        | Held and shown by IdP.       | Verifiable Presentation, containing Verifiable Credentials issued by any relevant authorities.
| Client ID                 | IdP-assigned. | URL of the Verifiable Presentation.
| Authorization Server      | Identity Provider (IdP), known by the Relying Party before creating the request.   | Chosen by user during authentication. Examples: user's own identity wallet, or a web application.
| Authorization endpoint    | Contains the Identity Provider (IdP)'s hostname. Either static or discovered through OIDC Discovery. | Custom scheme (`didconnect:`).
| Token endpoint (when applicable)           | Contains the Identity Provider (IdP)'s hostname. Either static or discovered through OIDC Discovery. | Provided as `token_uri` parameter in the authorization response. (**WIP**.)
| Userinfo endpoint           | Managed by IdP. Either static or discovered through OIDC Discovery.   | Contained in `id_token` (`userinfo` claim).
| Public key for id_token's signature by client | Either static or discovered through OIDC Discovery. | Listed in DID Document (DDO).
| Client's whitelisted redirect_uri endpoints | Registered and checked by the IdP. | Announced as a claim in the Client ID's Verifiable Presentation.
| Supported flows | Either static or discovered through OIDC Discovery. | Negotiated during authentication (**WIP**).

## General flow

The OIDC specification defines the following general flow:

1. Client registers (once).
2. Client presents an authentication request in the form of a `didconnect://auth?...` URI.
3. Authorization Server (AS) authenticates the Resource Owner (for example, if the AS is a mobile app, the user should unlock the app) and obtains their consent.
4. Depending on the selected flow, AS generates a combination of identity token, grant code, and/or access token, then provides them to the client's `redirect_uri` contained in the request, following the `response_mode` parameter (form, post or fragment).
5. The client receives the tokens and/or code.
6. *Authorization flow only.* Client gets the URL of the token endpoint and uses it to request the `id_token` and/or `access_token`.
7. If `id_token` was provided, client checks its signature and reads user's identity from the token as a JWT.
8. If a `userinfo` endpoint is available, client queries it, providing the access token for authentication.

Steps for the DIDConnect profile are detailed in the next sections.

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
| `response_type` | Optional | Depends on the grant flow. Each flow is described by a combination of `id_token`, `token`, `code`, as defined by OIDC.
| `response_mode` | Optional | How the AS must issue the tokens. Must be one of `query`, `fragment`, `form_post`.

### 3. Obtaining user consent from AS

When a user's Authorization Server receives a request from the application's client_id, it must treat `client_id` as an URL and fetch the Verifiable Presentation (VP) from it. Then, the AS must check the following conditions:

- The VP's proof is valid against its "holder" attribute, and the VP is within validity date.
- The `redirect_uri` value from the request matches one of the values present in the VP's valid credentials.

Then, the AS should request user's consent after showing the user all relevant claims, based on valid credentials.

Note: For `redirect_uri` and claims, "valid credentials" are credentials with a valid proof, within validity dates, and whose subject ID is the same as the VP's holder.

### 4. Token generation

See OpenID Connect specification.

### 5. Getting the tokens and/or authorization code

See OpenID Connect specification.

### 6. Accessing the tokens (if applicable)

See OpenID Connect specification. Token endpoint URI is taken from the `token_uri` parameter contained in the response.

### 7. Verification of user's identity

The endpoint listening on `redirect_uri` receives the following query parameters (as query parameters, HTML fragment or form post depending on the requested response mode):

| Parameter      | Status   | Description
|----------------|----------|------------
| `id_token`     | Required | A JSON Web Token (JWT) containing the user's DID.
| `state`        | Optional | The original `state` from the authentication request, if any.
| `access_token` | Optional | A Bearer token authorizing access to information about the user

The JWT contained in the `id_token` provides the following claims:

| Parameter | Description
|-----------|------------
| `sub`     | The user's DID.
| `iss`| The identifier (resolved from the DID Document) of the public key that is signing the JWT. The JWT is only valid if the key is authorised for that DID and if the JWT is correctly signed by that public key.
| `aud`| The requester's DID. The JWT is only valid if that value matches the value of `client_id` in the request.
| `state`| The state present in the request, to avoid replay attacks.
| `iat`| The date the JWT was issued at. You should check the JWT is not too old, otherwise it might be insecure.
| `exp`| The date until which the JWT is valid. You should check that the JWT is not expired, otherwise it might be insecure.
| `at_hash`| The hash of the `access_token`.
| `c_hash` | The hash of the `code`.
| `userinfo` | The URL of the Resource Server where a user's Verifiable Presentation is available.

To properly verify the user's identity, the client must:

- Verify the signature of id_token JWT against the public key identified in `iss`.
- Make sure the JWT's issuer is an authorized key of the JWT's subject (i.e. the user's DID), by matching it against the DID Document.

#### Note for GID
Currently, KayTrust's "GID" DID method does not support DID Documents. Until it does, the steps to validate a public key are the following:

1. Compute the user's Proxy address from the DID
2. Call `proxy.owner()` to get the Identity Manager's address, or use a well-known Identity Manager address for legacy DIDs.
4. Call `im.hasCap(proxy, device, "auth")` and verify the result is `true`. To support Levels of Assurance, replace "auth" with the expected Level of Assurance.

### 8. Accessing the user's protected resources (when applicable)

If an access token was provided, the client can access user's information by querying the URL referenced as `userinfo` in the id_token, and provide the access token as an `Authorization` header, as described in [OpenID Connect specification](https://openid.net/specs/openid-connect-core-1_0.html#UserInfoRequest).

The response is a Verifiable Presentation. The presentation's holder must be the user's authenticated DID. The presentation contains a `proof` attribute that allows the client to verify the holder's consent before using the included credentials.


## Examples of AS

Examples of Authorization Servers implementations include:

- **Mobile or desktop applications.** Registration to the `didconnect:` scheme is specific to the operating system.
- **Web applications.** Registration to the `didconnect:` scheme is done using `registerProtocolHandler()` on the web browser.

## Current limitations

### Flow support uncertainty

This specification describes the use of a custom URI scheme to express authorization requests. This makes it possible for any registered application on the user agent to handle requests. This approach, similar to that of the `mailto:` and `tel:` URI schemes, implies the client app does not know what AS implementation will actually respond to authorization requests, and what grant flows it will support.

This uncertainty represents both a risk and added out-of-band work on the client side to try and guess what flow to request. One approach is to determine the type of AS based on heuristics.

One possible improvement would be to use negotiation: the client would announces the flows it supports, and the AS makes a decision based on its own capabilities. This is similar to content type or language negotiation in HTTP.

The negotiation takes place in two steps:

1. Client announces in the authorization request the grant flows it supports, as a semicolon-separated list of supported response types.
2. AS selects a grant flow according on its own capabilities.

TODO: Details and examples.