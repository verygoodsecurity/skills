---
name: vgs-sftp-proxy-onboarding
description: Plan and provision VGS SFTP Proxy routes in SANDBOX with agent-executed vgs-cli, including entitlement checks, file selectors, route YAML, and read-back validation.
---

# VGS SFTP Proxy onboarding

Use with `vgs-agentic-onboarding`. First read the official
[SFTP Proxy](https://docs.verygoodsecurity.com/vault/batch-file-transmission/sftp-proxy)
and [SFTP Routes](https://docs.verygoodsecurity.com/vault/batch-file-transmission/sftp-proxy/sftp)
pages. SFTP Proxy requires product access; record missing enablement as a VGS
owner blocker rather than attempting a workaround.

## Required decisions

Confirm direction, upstream SANDBOX hostname and port, authentication owner,
file-path match, file format, sensitive field selectors, alias format, storage
mode, and redaction/reveal operation. Never put an SFTP password or private key
in the plan, route YAML, Git, or conversation.

## Route workflow

- Start from [assets/sftp-route.yaml](assets/sftp-route.yaml), which is a
  sanitized creation template derived from an exported route.
- Replace every placeholder and revalidate the shape against current official
  docs and `vgs apply routes --help` before proposing it.
- A creation payload must not contain route IDs, entry IDs, `created_at`, or
  `updated_at`. Those fields turn an exported document into an update or carry
  server-generated state.
- Use generic `vgs apply routes`; do not use `generate mft-route` or
  `apply tenant-resources` as an SFTP substitute.
- Preview the exact YAML digest, execute only after approval, then read the
  route back and verify `protocol: sftp`, endpoints, port, filter phase,
  operation, selector, and SANDBOX tenant.

Validate with a synthetic file containing no customer PII. Treat route
existence as configuration evidence, not proof that file transfer or
transformation works.
