# Behaviour: Resource Servers

_(c) AMWA 2019, CC Attribution-NoDerivatives 4.0 International (CC BY-ND 4.0)_

## Public keys

Resource Servers SHOULD maintain a cache of public keys in order to minimise the time to respond to authorized requests.

Resource Servers SHOULD seek to fetch public keys from one Authorization Server in the deployment at least once every
hour, altering their retrieval time by a value between 0-60 seconds to avoid overloading the Authorization Server due to
Resource Servers synchronising their retrieval time.

If a Resource Server is unable to contact an Authorization Server, the Resource Server SHOULD attempt to contact another
Authorization Server from the discovered list until this either succeeds or the list is exhausted. If no Authorization
Servers respond as expected the Resource Server MUST implement a randomised back-off mechanism to avoid overloading the
Authorization Servers.

If a Resource Server is unable to contact an Authorization Server, the Resource Server SHOULD assume currently held
public keys remain valid until it is able to re-establish a connection to an Authorization Server.

Resource Servers SHOULD attempt to verify tokens against all keys presented at the Authorization Server's public key
endpoint. All valid JWK's SHOULD be tried until the token is verified or until no keys are left.

Where a Resource Server has no matching public key for a given token, it SHOULD attempt to obtain the missing public key
via the the token `iss` claim as specified in [RFC 8414][RFC-8414] section 3. In cases where the Resource Server needs
to fetch a public key from a remote Authorization Server it MAY temporarily respond with an HTTP 503 code in order to
avoid blocking the incoming authorized request. When a HTTP 503 code is used, the Resource Server SHOULD include an
HTTP `Retry-After` header to indicate when the client may retry its request.

If the Resource Server fails to verify a token using all public keys available it MUST reject the token.

Resource Servers SHOULD NOT re-fetch public keys from Authorization Servers when validating token signatures unless
none of the public keys which they hold in their cache can be used to validate the token.

## Accessing Protected Resources

Access Tokens are passed to protected endpoints via the HTTP Authorization Request Header field, as per Section 2 of
[RFC 6750][RFC-6750].

When a protected resource receives a token it MUST validate the signature and claims of the token.

If a protected resource request does not include authentication credentials or does not contain an Access Token
that enables access to the protected resource, the resource server MUST reject the request with the appropriate
HTTP error code and include the HTTP `WWW-Authenticate` response header field using the auth-scheme value "Bearer",
as per Section 3 of [RFC 6750][RFC-6750].

For example:
*   A request without authentication or one including an expired Access Token will receive a 401 (Unauthorized) response.
*   A request that is not permitted by the values of the `x-nmos-*` claims, as per the
    [Access Tokens](Behaviour%20-%20Access%20Tokens.md) section, will receive a 403 (Forbidden) response.

Resource Servers are strongly RECOMMENDED to avoid the use of HTTP Redirects on API endpoints which require
Authorization. Responding directly to clients avoids redirection to an endpoint which might
exhibit different Authorization claim requirements.

### Operation with WebSocket

Where OAuth 2.0 is to be used with WebSocket clients SHALL provide the Access Token in the HTTP GET request that
initiates the Websocket handshake defined in [RFC 6455][RFC-6455] in the same manner as a normal HTTP request as
described in [Accessing Protected Resources](#accessing-protected-resources).

The Resource Server SHALL validate such tokens in the same manner as it would for a normal protected HTTP resources
using the HTTP Authorization Request Header.

Further to this, due to limitations in the native JavaScript WebSocket API, clients MAY pass the OAuth 2.0 Access Token
in the query parameters of the request URL during the handshake, using the `access_token` key, as described in
[RFC 6750][RFC-6750]. This is only for situations in which it is not feasible to pass the token in the HTTP
Authorization Header.

```
https://www.example.com/ws?access_token=mF_9.B5f-4.1JqM
```

The Resource Server MUST support the parsing of Access Tokens in both the Authorization HTTP Header and the query
parameter. The Resource Server SHALL NOT upgrade the connection to a WebSocket if the token is deemed invalid.

## Validation of Access Token

Resource Servers MUST validate the signature and Header of the JWT token in line with Section 5 of [RFC 7515][RFC-7515].
The certificate containing the public key for verification is found at the public key endpoint of the Authorization
Server that issued the token. This server is identified using the issuer (`iss`) claim within the token.

The value of the claims within the payload of the JWT must also be validated, in line with the
[Tokens](Behaviour%20-%20Access%20Tokens.md) section. The request MUST be rejected if:
*   the `iat` claim is greater than the current UTC time.
*   the `exp` claim is less than the current UTC time.
*   the Resource Server does not identify itself with the `aud` claim

### Path Validation

Resource Servers MUST verify that the URL path component requested matches one which is permitted by the token, ignoring any URL query parameters. The
following rules MUST be followed when confirming a match.

| Path | Requirements |
| ---- | ------------ |
| `/`  | Always allow 'read' permission with no authorization checks. This includes implicit permission to this path with or without a trailing slash |
| `/x-nmos` | Always allow 'read' permission with no authorization checks. This includes implicit permission to this path with or without a trailing slash |
| `/x-nmos/<api name>` | The token MUST include either an `x-nmos-*` claim matching the API name, a `scope` matching the API name or both in order to obtain 'read' permission. This includes implicit permission to this path with or without a trailing slash |
| `/x-nmos/<api name>/<api version>` | The token MUST include either an `x-nmos-*` claim matching the API name, a `scope` matching the API name or both in order to obtain 'read' permission. This includes implicit permission to this path with or without a trailing slash |
| `/x-nmos/<api name>/<api version>/<path>` | The token MUST include an `x-nmos-*` claim matching the API name and the path, in line with the method outlined in [Tokens](Behaviour%20-%20Access%20Tokens.md) |

## Interaction with other NMOS APIs

Any further token validation requirements based on the specific NMOS API being accessed is highlighted in [BCP-003-02][BCP-003-02].

## Audit Requirements

Resource Servers MUST securely provide a log of each request which they authorize or reject. Logs MUST include an
accurate timestamp and details of the token passed to them in the request. Logs MUST NOT contain sensitive information
such as secrets.


[RFC-6455]: https://tools.ietf.org/html/rfc6455 "The WebSocket Protocol"

[RFC-6750]: https://tools.ietf.org/html/rfc6750 "The OAuth 2.0 Authorization Framework: Bearer Token Usage"

[RFC-7515]: https://tools.ietf.org/html/rfc7515 "JSON Web Signature (JWS)"

[RFC-8414]: https://tools.ietf.org/html/rfc8414 "OAuth 2.0 Authorization Server Metadata"

[BCP-003-02]: https://specs.amwa.tv/bcp-003-02 "Authorization in NMOS Systems"
