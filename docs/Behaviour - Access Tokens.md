# Behaviour: Access Tokens

_(c) AMWA 2019, CC Attribution-NoDerivatives 4.0 International (CC BY-ND 4.0)_

The Access Token type returned MUST be of the `Bearer_token` type specified in [RFC 6750][RFC-6750].

The Access Token MUST be a JSON Web Signature (JWS) as defined by [RFC 7515][RFC-7515]. JSON Web Algorithms (JWA) MUST
NOT be used.

The JWS MUST be signed with `RSASSA-PKCS1-v1_5 using SHA-512`, meaning the value of the `alg` field in the token's JOSE
(JSON Object Signing and Encryption) header (see [RFC 7515][RFC-7515]) MUST be set to `RS512` as defined in
[RFC 7518][RFC-7518]. An example JOSE header would be:

```json
{
  "typ": "JWT",
  "alg": "RS512"
}
```

## Registered Claims

Registered claims are defined in the authorization JSON Web Token specification in [RFC 7519][RFC-7519], but their
specific usage is left to the application. NMOS API Clients and Servers implementing this specification MUST employ the
restrictions on claims outlined below, in addition to implementing tokens as specified in [RFC 7519][RFC-7519].

### iss
_Identifies principal that issued the JWT_

The `iss` (issuer) claim MUST be included in the token. The claim MUST contain a URL value that identifies the
Authorization Server that issued the JWT as specified in section 2 of [RFC 8414][RFC-8414]. The URL host value
SHOULD match one entry in the common name field or alternate names fields in all TLS certificates used by the
Authorization Server.

### sub
_Identifies the subject of the JWT_

The `sub` (subject) claim MUST be included in the token. This claim MUST contain a unique identifier assigned to the
end-user by the user authorization system. For example, this may be a username in a Single Sign-On (SSO) system, or the
email address of the user.

For the authorization code grant, the subject typically identifies the authorized accessor for which the Access Token
is being requested.

### aud
_Identifies the recipients of the JWT_

The `aud` (audience) claim MUST be included in the token. This claim MUST contain a JSON array of case-sensitive
strings, each containing a StringOrURI value that identifies the recipients that the JWT is intended for. It is
RECOMMENDED that the JSON array contains the fully resolved domain names of the intended recipients, or a domain
name containing wild-card characters in order to target a subset of devices on a network. Such wild-carding of domain
names is documented in [RFC 4592][RFC-4592].

If the `aud` claim is present and does not match the fully resolved domain name of the resource server, the Resource
Server MUST reject the token.

URI based `aud` claims MUST NOT include port numbers, URL paths or URL query parameters. Resource Servers MUST be capable
of interpreting both URI based `aud` claims which are prefixed by a scheme (such as `https://`) and string based `aud`
claims which specify only a domain name.

### exp
_Expiration time of the token_

The `exp` (expiration) claim MUST be included in the token. This is defined in [RFC 7519][RFC-7519] as being a JSON
NumericDate field, which uses the UTC epoch. This is in contrast to the TAI epoch used elsewhere within the NMOS APIs,
so implementers should take care to ensure they are using the correct epoch.

API implementations MUST reject a token where the `exp` claim value is less than the current UTC time. Refer to the
[Token Lifetime](#access-token-lifetime) Section for guidance on setting this value.

### iat
_Token issued at time_

The `iat` (issued at) claim MAY be included in the token. As with the `exp` claim this claim uses UTC. API
implementations MUST reject a token where the `iat` claim is greater than the current UTC time. Authorization Servers
SHALL issue a token with an `iat` claim that is equal to the UTC time at which the token is issued.

### client_id
_Client ID that requested the token_

The `client_id` claim MUST be included in the token, unless the `azp` claim is present in which case it is optional.
This claim corresponds with the client identifier of the OAuth 2.0 client that requested the token. The value of this
claim MUST exactly match the full client identifier to which the token was granted.

### azp
_Client ID that requested the token_

The `azp` claim MUST be included in the token when the `client_id` is omitted. The `azp` claim SHOULD NOT be included
in the token when the `client_id` is included in order to minimise token size. This claim is included in order to
support Authorization Servers which support the OpenID Connect specification. This claim corresponds with the client
identifier of the OAuth 2.0 client that requested the token. The value of this claim MUST exactly match the full client
identifier to which the token was granted.

### scope
_List of scopes associated with the token_

The `scope` claim SHOULD be included in the token, if the authorization request included a scope parameter, as per
Section 2.2.2 of the draft RFC [Profile for OAuth 2.0 Access Tokens][oauth-access-token-jwt]. The value of the "scope"
claim is a JSON string containing a space-separated list of scopes associated with the token, in the format described
in Section 3.3 of [RFC6749][RFC-6749].

The value of each scope corresponds to the namespace identifier for a given NMOS
API (e.g. "registration", "query", etc.) to which the user is requesting access and determines how the private
`x-nmos-*` claims, detailed below, are populated.

Presence of a `scope` matching an NMOS API grants implicit read only access to some API base paths as specified in
[Resource Servers](Behaviour%20-%20Resource%20Servers.md).

### nbf
_Not before time_

The `nbf` (not before) claim MAY be included in the token.  The "nbf" (not before) claim identifies the time before 
which the JWT MUST NOT be accepted for processing.  The processing of the "nbf" claim requires that the current 
date/time MUST be after or equal to the not-before date/time listed in the "nbf" claim. 
This is defined in  [RFC 7519](https://tools.ietf.org/html/rfc7519 "JSON Web Token (JWT)").

## Private Claims

[RFC 7519][RFC-7519] allows for "private claims". The following claims are used to identify the API specification(s) a given
token is used for.

### x-nmos-*
_Contains information particular to the NMOS API the token is intended for_

One or more `x-nmos-*` claims SHOULD be included in the token. The claim name begins `x-nmos-` followed by the namespace
identifier found in the URL for that given API (e.g. "registration", "query", etc.).

Presence of an `x-nmos-*` claim matching an NMOS API grants implicit read only access to some API base paths as
specified in [Resource Servers](Behaviour%20-%20Resource%20Servers.md).

The value of the claim is a JSON object, indicating access permissions for the API. An omitted `x-nmos-*` object
indicates that no access is permitted to the namespace-identified API beyond what may be granted by the presence of
a matching `scope`.

#### The Access Permissions Object
The value of each `x-nmos-*` claim is the access permissions object for the given user for that specific API. The access
permission object consists of `key:value` pairs in which the `key` denotes a specific permission. General permissions
include:
*   **write** - ability to insert a new, or edit an existing, resource. This takes the form of *POST*, *PUT*,
    *PATCH* and *DELETE* HTTP request verbs.
*   **read** - ability to retrieve the resource. This takes the form of *GET*, *OPTIONS* and *HEAD* HTTP request
    verbs.

If a permission has not been granted to a specific user, the permission key MUST be omitted, to minimise the
token size. If no permissions are given for a specific API, the `x-nmos-*` claim relating to that API key MUST be
removed to ensure token size is minimised.

If a permission of "write" is given in the access permission object, and the intention is that the given user
also has "read" access to the specific API, a permission key of "read" MUST also be present in the access permissions
object.

Individual AMWA specifications MAY specify additional permissions keys specific to the individual API. These MUST be
registered in the [AMWA NMOS Parameter Registers][AMWA NMOS Parameter Registers]. It is recommended that new
permissions keys are only introduced alongside a new version of a given AMWA NMOS API in order to minimise the risk of
incompatibility.

The `value` corresponding to each permission key is a JSON array containing URL path specifiers that the user is
permitted to perform the specific request against. The asterisk wildcard ('`*`') is permitted in the path
specifiers.

A '`*`' wildcard can be used to replace zero or more characters (this is equivalent to the `.*` regex pattern). The
wildcard may encompass multiple path segments, may be used multiple times, and may appear at any position in the path
permission (e.g. `single*` or `single/senders/*/constraints` would both match with
`/x-nmos/connection/v1.1/single/senders/ea388089-9ffb-4a81-b109-a19da845b3b6/constraints`). It is RECOMMENDED that the
wildcard is used in place of NMOS resource identifiers, such as UUIDs, to minimise token size. Specific identifiers MAY
be used in path specifiers if defining access to a small number of resources (such as a single node or device.)

Resource Servers MUST apply [URL Normalization][URL Normalization] to request URLs in order to remove '`..`' path
segments, which could otherwise introduce vulnerabilities. For example, the path
`/x-nmos/connection/v1.1/single/../bulk` MUST NOT be permitted by `single/*` path permission.

For NMOS API URL's that follow the standard pattern of:

`<api_proto>://<hostname>:<port>/x-nmos/<api type>/<api version>/`

all path specifiers MUST start _after_ the API version and its trailing slash. For example, authorizing a user to read
all _single resources_ and modify the configuration of all  _single senders_ at a Connection Management interface would
result in a claim of:

```JSON
"x-nmos-connection": {
  "read": ["single/*"],
  "write": ["single/senders/*"]
}
```

#### Example `x-nmos-*` Claims
Examples of `x-nmos-*` claims are shown below. This would permit querying an IS-04 Registry, and modifying
_single_ connections via an IS-05 connection management interface, but would not permit the user to register a resource
with an IS-04 Registry.

```json
"x-nmos-registration": {
  "read": ["*"]
},
"x-nmos-query": {
  "read": ["*"],
  "write": ["subscriptions/*"]
},
"x-nmos-connection": {
  "read": ["*"],
  "write": ["single/*"]
}
```

Authorization Servers MAY only have support for certain NMOS API specifications, and MAY only support certain versions
of such APIs.

## Size Considerations

While [RFC 7519][RFC-7519] does not prescribe a maximum size for an OAuth 2.0 JSON Web Token, it should be noted that
these tokens are typically used within an HTTP header. While HTTP does not define a header size limit, 8KByte is a
common limitation in HTTP server implementations. Specification writers should be mindful of this when designing API
claims, and ensure that enough space is available in the header for both the token and all other HTTP headers.

## Example Access Token Claim Set

```json
{
  "iss": "https://auth.example.com",
  "sub": "username@example.com",
  "aud": ["https://node-*.example.com"],
  "iat": 1548779460,
  "exp": 1548783060,
  "scope": "registration query connection",
  "client_id": "hopy0dNRPNTiGJDqPfqYwGmw",
  "x-nmos-registration": {
    "read": ["*"]
  },
  "x-nmos-query": {
    "read": ["*"],
    "write": ["subscriptions/*"]
  },
  "x-nmos-connection": {
    "read": ["*"],
    "write": ["single/*"]
  }
}
```

## Access Token Lifetime

A given Access Token for an NMOS API may be used on more than one Node or Registry instance. If one of these Nodes or
Registries is compromised, it is possible for that entity to re-use it itself maliciously. While the precise duration
of token validity should be left to implementers and administrators based on the risk profile, these tokens SHOULD be
valid for no more than one hour.

However, if Access Tokens are too short-lived, the number of refresh requests to the Authorization Server for new
tokens starts to become a significant overhead, and any latency involved in using a token may cause it to become
invalid during transit. As such tokens SHOULD be valid for at least 30 seconds.

<p align="center">30 seconds < TOKEN LIFETIME < 1 hour</p>


[RFC-4592]: https://tools.ietf.org/html/rfc4592 "The Role of Wildcards in the Domain Name System"

[RFC-6749]: https://tools.ietf.org/html/rfc6749 "The OAuth 2.0 Authorization Framework"

[RFC-6750]: https://tools.ietf.org/html/rfc6750 "The OAuth 2.0 Authorization Framework: Bearer Token Usage"

[RFC-7515]: https://tools.ietf.org/html/rfc7515 "JSON Web Signature (JWS)"

[RFC-7518]: https://tools.ietf.org/html/rfc7518 "JSON Web Algorithms (JWA)"

[RFC-7519]: https://tools.ietf.org/html/rfc7519 "JSON Web Token (JWT)"

[RFC-8414]: https://tools.ietf.org/html/rfc8414 "OAuth 2.0 Authorization Server Metadata"

[oauth-access-token-jwt]: https://datatracker.ietf.org/doc/draft-ietf-oauth-access-token-jwt/ "JSON Web Token Profile for OAuth 2.0 Access Tokens"

[AMWA NMOS Parameter Registers]: https://github.com/AMWA-TV/nmos-parameter-registers

[URL Normalization]: https://tools.ietf.org/html/rfc3986#section-6 "RFC 3986 URI Generic Syntax"
