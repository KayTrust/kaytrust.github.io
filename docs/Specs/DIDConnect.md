# OpenID Connect Profile for self-sovereign identity (DIDConnect)

## Introduction

This document explains how to authenticate a user against their self-sovereign identity. This method uses [OpenID Connect (OIDC)](https://openid.net/developers/specs/), which itself builds upon the [OAuth 2.0 framework](https://tools.ietf.org/html/rfc6749).

DID Connect uses OIDC in a way that enables Self-Sovereign Identity (SSI) and favours decentralization, by combining it with DIDs and, optionally, Verifiable Credentials. That makes it different from traditional provider-centric flows in a few aspects, as detailed below.

| Aspect                    | Provider-Centric Identity          | Self-Sovereign Identity
| ------------------------- | ------------------------- | -----------------------
| Authorization request URIs| Use the Identity Provider (IdP)'s hostname | Use `didconnect:` scheme.
| Authorization Server      | Identity Provider (IdP), known before when creating the request.   | Selected dynamically by user. Examples: user's own identity wallet, or a web application.
| Subject ID                | Assigned by IdP.         | User's DID.
| Client registration flow  | Client registers against IdP (must be authorized by IdP). Client ID is an opaque string.           | No authorization needed by a central entity for acting as a client. Client exposes a Verifiable Presentation, containing Verifiable Credentials issued by relevant authorities. Client ID is the URL of that presentation.
| Userinfo endpoint           | Managed by IdP. Endpoint URL is either static or discovered through OIDC Discovery.   | Provided dynamically: endpoint URL is present in id_token ("userinfo" claim).
| Public key for id_token's signature by client | Either static or discovered through OIDC Discovery. | Provided by DID Document (DDO).
| Client's whitelisted redirect_uri endpoints | Registered and checked by the IdP. | Announced as a claim in the Client ID's Verifiable Presentation.

## General flow

The OIDC specification defines the following general flow:

1. The clients registers (only once).
2. The client presents an authentication request in the form of a `didconnect://auth?...` URI.
3. The Authorization Server authenticates the user (for example, if the AS is a mobile app, the user should unlock the app) and obtains their consent.
4. Depending on the grant flow, the Authorization Server generates a combination of identity token, grant code, and/or access token, then provides them to the client's `redirect_uri` contained in the request, according to the `response_mode` parameter (form, post or fragment).
5. The client receives the tokens and/or code.
6. If a code was received, the client gets the URL of the token endpoint from the `token_uri` parameter contained in the response and uses it to get the id_token and/or access_token.
7. If an id_token was provided, the client verifies the user's identity from it.
8. If a `userinfo` claim was present in the id_token, the client extracts the userinfo endpoint URL from it and queries it, providing the access token for authentication. The response is a verifiable presentation held by the user.

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

See OpenID Connect specification. TODO: Describe how token endpoint is read dynamically from response.

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

## Limitations and possible improvements

### Flow support uncertainty

This specification describes the use of a custom URI scheme to express authorization requests. This makes it possible for any registered software on  the user agent to handle requests. This approach, similar to that of the `mailto:` and `tel:` URI schemes, implies the client app does not know what AS will actually respond to authorization requests, and what grant flows it will support.

This uncertainty represents both a risk and added out-of-band work on the client side to try to guess what flow to request. One approach is to determine the type of AS based on heuristics. Another approach is to present multiple requests and let the user or the user agent make a decision.

A possible improvement (see Appendix below) would be for a negotiation mechanism to be defined, where the client would announce the flows it supports, and let the AS make a decision based on its own capabilities. This would be similar to content type or language negotiation in HTTP.




# Appendix: Ideas around Authorization Server diversity and flow negotiation

This appendix describes a method for an OAuth client to dynamically negotiate with the AS the OAuth grant flow to be used, similar to content type or language negotiation in HTTP. This is useful in situations where the client can't know the flows that will be supported by the AS. One such example is a DIDConnect flow.

The negotiation is in two steps:

1. The client announces in the authorization request the grant flows it supports, as a semicolon-separated list of supported response types.
2. The AS selects a grant flow according on its own capabilities.

TODO: Details and examples.