# Security policy
<!-- rev:001 (RFC 3339) 2026-10-17T22:39:55Z -->

## Reporting a vulnerability

**Do not open a public issue, discussion or pull request for a vulnerability.** Report it privately through GitHub's
"Report a vulnerability" (Security tab → Advisories) on the affected repository. Include what is affected (repository,
endpoint or package and version), how to reproduce it, and the impact you expect. Never include real customer data or
working credentials; describe them instead.

You will receive an acknowledgement, and the fix is tracked in a private advisory until it is released. Please give us
a reasonable time to release a fix before any public disclosure.

## Scope

Every repository of the `inovacc-core` organization, the public API at `https://api.inovacc.dev` and its hosts
(`auth.inovacc.dev`, `mcp.inovacc.dev`, `nexum.inovacc.dev`, `developer.inovacc.dev`), and the official SDKs
(`github.com/inovacc/inovacc-<language>`).

## Rules for contributors

- Never commit a secret, a real token or a credential-shaped literal, not even as a test fixture (build such strings at
  runtime); secret scanning runs on every pull request.
- Never print or log a secret, a token or personal data; cite a location with the value redacted.
- Encoding is not hiding: base64 or hex of a secret is still a secret.
