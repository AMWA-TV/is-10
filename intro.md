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
