# Behaviour: Authorization Servers

_(c) AMWA 2019, CC Attribution-NoDerivatives 4.0 International (CC BY-ND 4.0)_

## Keys

### Authorization Server Public Keys

Authorization Servers MUST provide all public keys used for signing tokens at a non-protected endpoint, the location
of which will correspond with the value of the "jwks_uri" property at the server metadata endpoint, for example
**/jwks**.

The format of the keys MUST be the JSON Web Key (JWK) format described in Section 4 of [RFC 8414][RFC-8414] and
MUST form a JSON Web Key Set (JWKS) described in Section 5 of [RFC 8414][RFC-8414]. Authorization Servers MAY
present more than one key within a JSON Web Key Set, with each key being an entry in the "keys" array.

Where multiple Authorization Servers exist in a single deployment they SHOULD each host a copy of each other's public
keys in order to prevent Resource Servers having to make requests to every instance.

### Changing Keys

When transitioning to a new public/private key pair used for signing tokens an Authorization Server SHOULD provide
both the old and new public key at the public key endpoint until all tokens that may be verified by the old public key
would have expired. However, if a private key is known to be compromised, the Authorization Server MUST remove it
from the public key endpoint immediately.

Authorization Servers SHOULD provide new public keys at the public key endpoint for at least 2 hours before
issuing tokens signed by the corresponding private key to allow time for clients to cache the new public key.

## Client Registration

Authorization Servers MUST NOT grant tokens to unregistered clients.

Authorization Servers SHOULD support a manual mechanism for registering clients (e.g an HTML web form) allowing the
client type and redirect URIs to be provided.

Clients and Authorization Servers MUST provide for the OAuth 2.0 Dynamic Client Registration Protocol
[RFC 7591][RFC-7591] to allow dynamic registration of clients to the Authorization Server to allow for
zero-configuration setup.

Authorization Servers MAY additionally support the OAuth 2.0 Dynamic Client Registration Management Protocol
[RFC 7592][RFC-7592] to allow clients to check and modify existing registrations.

Authorization Servers MUST support authentication of dynamic client registrations via one or more of the mechanisms
below, which are listed in order of preference:
*   Presentation of an initial JSON Web Token (JWT) as suggested in Section 3 of [RFC 7591][RFC-7591]
*   Manual operator approval after unauthenticated registration, via the Authorization Server user interface or API

Where dynamic client registrations request the `client_credentials` grant type they MUST be subject to one of the
explicit authentication steps listed above. Authorization Servers MAY be configured to grant implicit authentication
where only the `authorization_code` grant type is requested.

After successful registration, an Authorization Server MUST generate a Client ID and, in the case of a confidential
client, a client secret. The Client ID MUST be at least 20 characters long, to help prevent brute-force attacks, and
MUST be unique to each client. Public clients MUST NOT be issued with a client secret, and Authorization Servers MUST
NOT accept client credentials as valid authentication for a public client, however the Authorization Server MAY issue
a client secret or other credentials for a specific installation of a native application client on a specific
device, in line with Section 10.1 of [RFC 6749][RFC-6749].

Client credentials MUST be returned to the client in the response of a successful dynamic client registration. The
method by which client credentials are passed to a client during manual registration is beyond the scope of this
specification.

AMWA NMOS Specifications MAY specify additional registration parameters where they require them. Where clients and
Authorization Servers that support OAuth 2.0 are used with these specifications they SHALL use these parameters in
all OAuth 2.0 registration mechanisms they support.

### Client Authentication

Authorization Servers MUST provide support client authentication at the token and revocation endpoints using the
following methods. Additional methods MAY also be supported.
*   HTTP Basic Authentication as per Section 2 of [RFC 2617][RFC-2617], and in the manner described in Section 2.3 of
[RFC 6749][RFC-6749]. This mechanism is identified by the client registration `token_endpoint_auth_method` of
`client_secret_basic`.
*   Digitally signed JSON Web Tokens in the manner described in Section 2.2 of [RFC 7523][RFC-7523]. This mechanism is
identified by the client registration `token_endpoint_auth_method` of `private_key_jwt`.

The security considerations outlined in Section 10.1 of [RFC 6749][RFC-6749] SHOULD be abided by with regards to client
authentication.

### Client Deregistration

Authorization Servers SHOULD support deregistration of clients in the case of a compromised client. The means by which
this client deregistration occurs is beyond the scope of this specification.

## Responding to Token Requests

Refer to [Token Requests](Behaviour%20-%20Token%20Requests.md) for information on supported OAuth 2.0 grants,
requesting Access Tokens and refreshing and revoking tokens.

## Audit Requirements

Authorization Servers MUST securely provide a log of each authorization and client registration which is performed,
including initial token generation and token refreshes. Logs MUST include an accurate timestamp and an identifier for
the user who authorized the action. Logs MUST NOT contain sensitive information such as secrets.


[RFC-2617]: https://tools.ietf.org/html/rfc2617 "HTTP Authentication: Basic and Digest Access Authentication"

[RFC-6749]: https://tools.ietf.org/html/rfc6749 "The OAuth 2.0 Authorization Framework"

[RFC-7523]: https://tools.ietf.org/html/rfc7523 "JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants"

[RFC-7591]: https://tools.ietf.org/html/rfc7591 "OAuth 2.0 Dynamic Client Registration Protocol"

[RFC-7592]: https://tools.ietf.org/html/rfc7592 "OAuth 2.0 Dynamic Client Registration Management Protocol"

[RFC-8414]: https://tools.ietf.org/html/rfc8414 "OAuth 2.0 Authorization Server Metadata"
