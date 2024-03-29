# Upgrade Path

_(c) AMWA 2019, CC Attribution-NoDerivatives 4.0 International (CC BY-ND 4.0)_

As is common with web APIs, over time changes will be made to support new use cases and deprecate old ways of working.
The NMOS APIs are no different, and have been designed to permit in-service upgrades across a facility which may be
running large amounts of equipment with support for different versions of these specifications.

API versioning is specified in the [APIs](APIs.md) documentation, with procedures for handling upgrades described
below.

## Requirements for Authorization Servers and OAuth Clients

Implementers of the Authorization API must support at least one API version, and may support more than one at a time.

Implementers of Authorization Servers and OAuth 2.0 Clients are strongly recommended to support multiple versions of the
Authorization API simultaneously in order to ease the upgrade process in live facilities.

NMOS Nodes act as both OAuth 2.0 Clients (when interacting with an IS-04 Registry) and as Resource Servers (when being
interacted with by an IS-05 connection management instance for example). The seperate functionality relating to these
distinct roles should be upgraded separately.

## Performing Upgrades

The following procedure is suggested for a live system which needs to migrate between API versions.

*   Upgrade Authorization API implementations to support the new API version, whilst providing backwards compatibility
to old versions active in the running system.
*   Upgrade OAuth 2.0 clients to support newer versions, which must support all Authorization API versions you are
currently using in your deployment. This should ensure OAuth 2.0 Clients are able to request tokens with any updated or
added claims.
*   Upgrade Resource Servers to support the new API version and token contents.
