# Definitions

_(c) AMWA 2019, CC Attribution-NoDerivatives 4.0 International (CC BY-ND 4.0)_

_See also the [NMOS Glossary](https://specs.amwa.tv/nmos/branches/main/docs/Glossary.html) for concepts regarding the [JT-NM
reference architecture](https://jt-nm.org/reference-architecture/), and definitions within RFCs._

The below definitions are based on the [OAuth 2.0 spec][RFC-6749].

## API

An HTTP/WebSocket Application Programming Interface as defined in an AMWA NMOS Specification (e.g. IS-04, IS-05,
  IS-06, etc.)

## Protected Resource

Any part of the API that is access-restricted. This may apply to read (e.g GET) or write (e.g POST) operations.

## Resource Server

The entity that is providing APIs containing protected resources, for example:

*   a Registry implementing IS-04 Registration and Query APIs
*   a Node implementing IS-04 Node API and IS-05 Connection API.

## Resource Owner / End-User

An entity capable of granting a client access to a protected resource. When the resource owner is a person, it is
referred to as an end-user.

## Authorization Server

The server hosting the Authorization API. It is responsible for the issuing of authorization codes and Access Tokens
to registered clients, after successfully authenticating the client and resource owner and obtaining authorization.

## Client

The entity, usually a Web Application, Native application, or SPA (Single Page Application) that is attempting to act on
the user’s behalf or access a Protected Resource. A client must register with the Authorization Server before being able
to request tokens. Once registered, the client is assigned client credentials in the form of a `client_id` and, for
confidential clients, a `client_secret`. Clients are usually deemed to be Confidential or Public based on their
ability to authenticate securely with the Authorization Server.

Within NMOS this may be:

*   a Node registering with a Registry using the IS-04 Registration API
*   a monitoring application querying Nodes using the IS-04 Query API
*   a connection control application using the IS-05 Connection API

## Access Token

A short-lived JSON Web Token (JWT) that may be used by a client to access Protected Resources on the Resource Server. It
consists of a JSON object in which each key:value pair is referred to as a claim.

## Bearer Token

A Bearer Token is passed from the Authorization Server to the client after successful authentication of the request.
Section 5.1. of [RFC 6749][RFC-6749] defines it as consisting of:
*   an `access_token` - this is the JWT used to access a protected resource.
*   an `expires_at` value - this coincides with the lifetime of the Access Token
*   a `token_type` - always "Bearer" within the scope of this document
*   a `refresh_token` (OPTIONAL) - this is present depending on the grant type. Refresh Tokens can be used to retrieve
further Access Tokens from the Authorization Server if the shorter-lived Access Token has expired.
*   a `scope` (OPTIONAL) - if the scope granted by the Authorization Server differs from the requested scope in the
authorization request, the granted scope must be included in the Bearer Token.


[RFC-6749]: https://tools.ietf.org/html/rfc6749 "The OAuth 2.0 Authorization Framework"
