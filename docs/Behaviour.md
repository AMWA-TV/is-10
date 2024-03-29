# Behaviour

_(c) AMWA 2019, CC Attribution-NoDerivatives 4.0 International (CC BY-ND 4.0)_

IS-10 defines behaviour for:

-   [Authorization Servers](Behaviour%20-%20Authorization%20Servers.md)
-   [Clients](Behaviour%20-%20Clients.md)
-   [Token Requests](Behaviour%20-%20Token%20Requests.md)
-   [Access Tokens](Behaviour%20-%20Access%20Tokens.md)
-   [Resource Servers](Behaviour%20-%20Resource%20Servers.md)


## Time Synchronization

This specification is dependent upon accurate time synchronization between all system components.

Authorization Servers, Resource Servers and Clients (or the hosts which they are deployed onto) MUST be capable of
synchronizing their clocks to an external source of time. It is STRONGLY RECOMMENDED that Network Time Protocol (NTP) is
supported for this purpose.

Deployment environments SHOULD include one or more providers of time synchronization data matching the requirements of
any chosen Authorization components.
