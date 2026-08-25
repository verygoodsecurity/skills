---
name: vgs-show-onboarding
description: Select and integrate the appropriate VGS Show SDK for web, iOS, Android, or React Native after SANDBOX proxy and alias setup.
---

# VGS Show onboarding

Identify the application platform before changing code. VGS Show is
application integration, not a VGS CLI mutation.

Read the shared plan's selected steps, detected project scopes, existing
evidence, and verified integration contracts first. Load and modify only the
selected client platform. For example, `react_native_integration` does not
touch web, native iOS, or native Android steps. Verify platform and backend
dependencies against the current repository rather than trusting a prior
`complete` label alone, then update only this step's status, non-secret
evidence, and timestamp.

- For React Native, iOS, or Android, inspect the matching current VGS Show SDK
  repository for an installable agent skill. The agent installs and uses the
  matching skill itself; the customer does not run installation commands.
- For web, or when an SDK skill is unavailable, use the guarded documentation
  fallback: read only current official VGS Show pages, including
  [Show.js configuration](https://docs.verygoodsecurity.com/vault/developer-tools/vgs-show/js/configuration)
  and its reference documentation, before implementing.
- Record the package or script version resolved by the customer project. Do not
  copy a version from stale onboarding prose.
- Require backend-controlled authorization, least-privilege reveal routes,
  masking by default, explicit copy behavior, and error states that do not leak
  aliases or raw values.
- Configure the SDK request using the verified route method, proxy path,
  content type, and JSON field structure from `integration_contracts`. If those
  values are missing or conflict with route read-back, block instead of
  inventing an application contract.
- Run the SDK repository's focused build, typecheck, and tests plus a dependency
  audit appropriate to the resolved application graph.

Use only synthetic SANDBOX aliases. Rendering a masked component proves UI
integration; it does not prove route authorization or reveal correctness.
