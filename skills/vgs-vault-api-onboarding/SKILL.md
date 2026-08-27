---
name: vgs-vault-api-onboarding
description: Configure SANDBOX Vault API access with agent-executed vgs-cli and guide aliasing, fingerprinting, and reveal integration without exposing credentials or raw sensitive values.
---

# VGS Vault API onboarding

Use with `vgs-agentic-onboarding`. Read the current official
[Vault API introduction](https://docs.verygoodsecurity.com/vault/developer-tools/apis/vault-api/introduction),
[authentication](https://docs.verygoodsecurity.com/vault/developer-tools/apis/vault-api/authentication-and-authorization),
and [Alias API](https://docs.verygoodsecurity.com/vault/developer-tools/apis/vault-api/alias)
pages before proposing scopes or application behavior.

## Decisions

Confirm persistent versus volatile storage, alias format, classifiers,
fingerprinting, whether exact-match deduplication is required, and whether raw
value reveal is genuinely required. Default to `aliases:write`; add
`aliases:read` only for a confirmed reveal use case. Reveal is disabled by
default and may require VGS enablement.

## CLI-owned setup

- Use current `vgs generate service-account --help` to create an editable
  service-account resource scoped to the exact SANDBOX tenant.
- Replace template scopes with the minimum confirmed Vault API scopes and
  validate the resource before approval.
- The agent executes `vgs apply service-account` using the core skill's
  protected-output procedure. It never reads the returned client secret.
- Read back only the account identifier, tenant restrictions, and scope names.

The CLI configures access; application calls to `/aliases` are not CLI
operations. Implement those calls only when application-code changes are
separately in scope. Never send real PII during onboarding. Validate with
synthetic values and do not reveal those values into the assistant context.
