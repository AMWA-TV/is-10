# Behaviour: Clients

## Discovery

Clients MUST be capable of discovering the location of an Authorization Server using DNS Service Discovery. Clients
MAY also permit manual configuration of one or more Authorization Servers.

When first contacting an Authorization Server, a Client MUST identify the location of API endpoints using the
Authorization Server Metadata resource as specified in [RFC 8414][RFC-8414]. Clients MUST NOT assume that every
Authorization Server instance on a network uses the same endpoint locations.

## Client Registration

Clients MUST be registered with the Authorization Server before initiating the OAuth 2.0 protocol as per Section 2
of [RFC 6749][RFC-6749]. Clients MAY assume that all discovered Authorization Servers on a network use the same
database of client registrations.

Clients SHOULD support a mechanism for configuring the information required for OAuth 2.0 registration (e.g a web-page
containing the required details).

Clients are RECOMMENDED to provide for the OAuth 2.0 Dynamic Client Registration Protocol [RFC 7591][RFC-7591] to allow
for zero-configuration setup. NMOS Nodes SHOULD be capable of registering dynamically in order to limit the
configuration associated with large numbers of Nodes coming online simultaneously. Clients MUST include populated
`client_name` and `scope` parameters when performing dynamic registrations. The `client_name` MUST be unique to each
instance of the software or hardware. It is RECOMMENDED that the `client_name` include the manufacturer's name, product
identifier and serial number.

Clients that do not register with an explicit `token_endpoint_auth_method` parameter will be registered as a
Confidential Client and issued with a client secret. Public clients MUST specify a value of "none" for this parameter
to notify the Authorization Server that they are a public client.

Clients which require use of the `client_credentials` grant type MUST support the following means of authentication
during the dynamic client registration process. All other Clients SHOULD support these authentication methods:
*   Presentation of an initial JSON Web Token (JWT) as suggested in Section 3 of [RFC 7591][RFC-7591]
*   Unauthenticated registration

Clients MUST be configurable to choose between the above authentication methods, which are listed in order of
preference. When an initial JWT authentication method is selected, Clients MUST provide the means to configure a custom
value for this initial JWT.

Clients MAY check and modify existing registrations with an Authorization Server as-per [RFC 7592][RFC-7592]. As this
is an optional feature of Authorization Servers, Clients MUST confirm that this is implemented as indicated by the
presence of the keys `registration_client_uri` and `registration_access_token` in a client registration response.

## Client Credentials

Confidential Clients SHOULD support authentication using a digitally signed JSON Web Token, to authenticate with the
Authorization Server's token and revocation endpoints in the manner described in Section 2.2 of [RFC 7523][RFC-7523].
This authentication method is identified as `private_key_jwt` in the `token_endpoint_auth_method` string during client
registration. Clients MUST be capable of hosting their own public JWKs at a location that corresponds to the value of
the registered `jwks_uri` used when registering with the Authorization Server. Clients MUST use the `jwks_uri` in
preference to `jwks` unless interacting with an Authorization Servers which supports Client Update Requests as per
Section 2.2 of [RFC 7592][RFC-7592], in which case they MAY choose to use `jwks`.

Confidential Clients MAY support HTTP Basic Authentication, as per Section 2 of [RFC 2617][RFC-2617], to authenticate
with the Authorization Server's token and revocation endpoints in the manner described in Section 2.3.1 of
[RFC 6749][RFC-6749]. This authentication method is identified as `client_secret_basic` in the
`token_endpoint_auth_method` string during client registration.

Confidential Clients are RECOMMENDED to prefer the `private_key_jwt` authentication method unless this is too
computationally expensive.

Client credentials SHOULD be stored by the client in a safe, permission-restricted, location in non-volatile memory in
case of a device restart to prevent duplicate registrations. Client secrets SHOULD be encrypted before being stored to
reduce the chance of client secret leaking.

## Client Endpoints

Clients implementing the Authorization Code grant MUST host at least a single endpoint, the Redirection Endpoint, as
described in Section 3.1.2 in [RFC 6749][RFC-6749], capable of receiving a request containing the authorization code.
This endpoint MUST be exactly identical to one of the entries in the array of redirect URI's registered during client
registration.

Redirect URIs MUST be complete (fully-qualified) and not use pattern-matching, as this makes them susceptible to
Redirect URI Validation Attacks, described in Section 4.1. in the
[OAuth 2.0 Security Best Current Practice][oauth-security-topics] document.

All registered redirect URI's for Clients SHOULD mandate use of TLS or other transport-layer security method.

Clients implementing the Authorization Code grant SHOULD provide the means to generate a valid redirect to the
Authorization Server's authorization endpoint, enabling users to initiate the Authorization process.

All OAuth 2.0 Clients SHOULD support PKCE and Public Clients MUST support PKCE as outlined in [RFC 7636][RFC-7636].

## Client Types

### Native Applications

Native applications (such as Desktop Applications) are regarded as Public Clients according to Section 2.1 of
[RFC 6749][RFC-6749].

Native applications SHOULD abide by the guidelines set out in OAuth 2.0 for Native Apps [RFC 8252][RFC-8252].

Native Applications MUST make authorization requests through external user-agents, such as a browser on the same
device.

Registered redirect URI's MUST comply with Section 7 of [RFC 8252][RFC-8252].

### User-Agent-based / Browser-based Applications

Browser-based applications (such as Single-Page Web Applications) are regarded as Public Clients according to
Section 2.1 of [RFC 6749][RFC-6749]. On the other hand, web applications that have a back-end server are capable of
keeping client credentials secure and therefore can be treated as Confidential Clients.

Browser-based applications SHOULD abide by the guidelines set out in
[OAuth 2.0 for Browser-Based Apps][oauth-browser-based-apps].

Browser-based applications communicating with an Authorization Server and Resource Server on the same domain are
RECOMMENDED to take advantage of using domain-specific, HTTP-Only cookies to store and exchange Access Tokens with
Resource Servers, due to their resilience to Cross-Site Scripting attacks, as stated in Section 6.1 of
[OAuth 2.0 for Browser-Based Apps][oauth-browser-based-apps].

Browser-based applications are NOT RECOMMENDED to store Access Tokens or Refresh Tokens in local storage due to their
susceptibility to Cross-Site-Scripting attacks, and instead rely on in-memory storage during the session.

## Requesting a Token

Refer to [Token Requests](Behaviour%20-%20Token%20Requests.md) for information on supported OAuth 2.0 grants,
requesting Access Tokens and refreshing and revoking tokens.

## Refreshing a Token

Clients SHOULD ensure Access Tokens are refreshed sufficiently in advance of their expiry. While the exact time will
depend on the implementation of the client and Authorization Server, it is RECOMMENDED to attempt a refresh at least 15
seconds before expiry (i.e the half-life of the shortest-lived token possible).

Clients MUST discard any old Refresh Tokens once a new Refresh Token is issued.

Clients MUST be capable of handling the situation in which the currently held Refresh Token has expired, and a new
authorization request process is required.

## Accessing Protected Resources

When accessing protected resources clients MUST include the Access Token in the request using the Authorization
Request Header Field method described in Section 2.1 of [RFC 6750][RFC-6750]. Clients MUST NOT use any of the other
methods specified in Section 2.0 of [RFC 6750][RFC-6750] when making requests via HTTP/S, except for the case noted in
[Resource Servers - Operation with WebSocket](Behaviour%20-%20Resource%20Servers.md#operation-with-websocket).

Access Tokens signed with a JSON Web Signature (JWS) are base64-url encoded and so MUST NOT be base64-url
encoded again before being sent in a request.

Clients MUST be capable of handling early revocation of tokens, which will be signalled by Resource Servers using HTTP
4xx error codes.
Further details on when Resource Servers will respond with particular codes is covered in
[Resource Servers - Accessing Protected Resources](Behaviour%20-%20Resource%20Servers.md#accessing-protected-resources).

[RFC-2617]: https://tools.ietf.org/html/rfc2617 "HTTP Authentication: Basic and Digest Access Authentication"

[RFC-6749]: https://tools.ietf.org/html/rfc6749 "The OAuth 2.0 Authorization Framework"

[RFC-6750]: https://tools.ietf.org/html/rfc6750 "The OAuth 2.0 Authorization Framework: Bearer Token Usage"

[RFC-7523]: https://tools.ietf.org/html/rfc7523 "JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants"

[RFC-7591]: https://tools.ietf.org/html/rfc7591 "OAuth 2.0 Dynamic Client Registration Protocol"

[RFC-7592]: https://tools.ietf.org/html/rfc7592 "OAuth 2.0 Dynamic Client Registration Management Protocol"

[RFC-7636]: https://tools.ietf.org/html/rfc7636 "Proof Key for Code Exchange by OAuth Public Clients"

[RFC-8252]: https://tools.ietf.org/html/rfc8252 "OAuth 2.0 for Native Apps"

[RFC-8414]: https://tools.ietf.org/html/rfc8414 "OAuth 2.0 Authorization Server Metadata"

[oauth-browser-based-apps]: https://datatracker.ietf.org/doc/draft-ietf-oauth-browser-based-apps/ "OAuth 2.0 for Browser-Based Apps"

[oauth-security-topics]: https://datatracker.ietf.org/doc/draft-ietf-oauth-security-topics/ "OAuth 2.0 Security Best Current Practice 13"
