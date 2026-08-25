---
name: vgs-https-proxy-onboarding
description: Provision and verify VGS HTTPS Proxy routes and redaction or reveal filters in SANDBOX through agent-executed vgs-cli.
---

# VGS HTTPS Proxy onboarding

Use with `vgs-agentic-onboarding`. Read the current official
[HTTPS Proxy](https://docs.verygoodsecurity.com/vault/http-proxy),
[filters](https://docs.verygoodsecurity.com/vault/http-proxy/filters), and
[outbound routes](https://docs.verygoodsecurity.com/vault/http-proxy/outbound-connection)
pages for the selected direction.

Confirm inbound or outbound direction, SANDBOX upstream hostname, path and
content-type conditions, request or response phase, operation, selectors,
alias format, storage, and application proxy ownership.

## CLI workflow

1. Fetch and preserve current routes before editing.
2. Use the reusable top-level `data:` route shape returned by `vgs get routes`.
   Do not pass the `generate http-route` resource envelope directly to
   `vgs apply routes`.
3. For creation, omit route and entry IDs. For update, preserve only the exact
   identifiers read from the selected tenant.
4. Include required REDACT or REVEAL entries in the approved route YAML; do not
   leave reveal filters as an assumed later Dashboard step.
5. Preview, approve, apply, and read back under the core execution contract.
6. For every dependent application step, populate `integration_contracts` from
   the approved route and read-back: route reference, method, exact proxy path,
   content type, and synthetic request and response JSON shapes. Verify that
   route selectors address those exact JSON fields; never invent or copy a
   contract from a different route.

Validate with synthetic SANDBOX traffic. Do not inspect access logs or captured
payloads in agent context. A pass-through route without the required operation
is incomplete.
