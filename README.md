# AMWA IS-10 NMOS Authorization Specification

[![Lint Status](https://github.com/AMWA-TV/nmos-authorization/workflows/Lint/badge.svg)](https://github.com/AMWA-TV/nmos-authorization/actions?query=workflow%3ALint)
[![Render Status](https://github.com/AMWA-TV/nmos-authorization/workflows/Render/badge.svg)](https://github.com/AMWA-TV/nmos-authorization/actions?query=workflow%3ARender)

<!-- INTRO-START -->

### What does it do?

- Allows an API server to accept or reject requests depending on what a client is authorized to do

### Why does it matter?

- Security in the control plane is essential
  - Best practice is to limit what clients can do

### How does it work?

- Control client provides credentials and gets an access token
  - Sends token with API requests
- Based on JSON Web Tokens and OAuth 2.0
- Encryption is a prerequisite (see BCP-003-01)

<!-- INTRO-END -->

## Getting started

There is more information about the NMOS Specifications and their GitHub repos at <https://specs.amwa.tv/nmos>.
