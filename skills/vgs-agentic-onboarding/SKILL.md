---
name: vgs-agentic-onboarding
description: Execute an approved VGS SANDBOX onboarding plan with vgs-cli, including tenant, route, and service-account setup, when the user wants the agent—not the user—to run CLI commands.
---

# VGS agentic onboarding

Use this skill only after needs discovery produced a confirmed
`VGS_ONBOARDING.md` and the user explicitly authorized implementation. This is
the execution skill for agent-run VGS CLI onboarding; do not route this workflow
through a command-guidance-only skill that requires user copy/paste.

The plan may be shared across Dashboard, backend, frontend, and mobile
implementations without assuming company roles. Before installing the CLI or
loading component skills, inspect `requested_platforms`,
`detected_project_scopes`, `selected_steps`, and `implementation_steps`.
Continue only when an incomplete `agent_vgs_cli` step is selected and its
dependencies have current non-secret evidence. Dashboard-only or
application-only selection does not authorize CLI execution.

Require the exact non-null `tenant_id` supplied or confirmed by the user and
`environment_confirmed: true` before authentication or mutation. Verify through
safe reads that the authenticated tenant is that identifier and is SANDBOX;
block on any mismatch and never infer an identifier from a name or prefix.

Read [references/execution-contract.md](references/execution-contract.md)
before any authenticated read or mutation. Load only the component skills
selected by the plan:

- `vgs-sftp-proxy-onboarding`
- `vgs-vault-api-onboarding`
- `vgs-https-proxy-onboarding`
- `vgs-show-onboarding`

If a selected component skill is not installed, install only that skill at
global scope so the customer repository remains unchanged:

```bash
DISABLE_TELEMETRY=1 npx skills add verygoodsecurity/skills --global --skill <COMPONENT_SKILL> --yes
```

Confirm that the selected global skill is available before continuing. Never use
a project-local installation, ask the user to run the command, or leave
`.agents/` or `skills-lock.json` in the customer repository. Use the guarded
fallback when global installation fails, would modify the project, or the
published skill is unavailable.

## Execution boundary

- Execute only SANDBOX work named in the approved artifact. Reject LIVE,
  unknown, or mismatched tenant environments.
- The agent runs every `vgs` command. Never ask the user to copy, paste, or type
  a CLI command.
- User interaction is limited to decisions, mutation approvals, completing an
  agent-started browser login, and retrieving secrets from an approved
  protected file that the agent never reads.
- Prefix commands with `VGS_CLI_SOURCE=vgs-agentic-onboarding/v1`.
- Treat installed `vgs --help` and nested help as the command contract. Official
  VGS documentation supplies product behavior. Never invent a flag, scope,
  route field, endpoint, or identifier.
- Never run access logs, debug mode, or traffic capture during onboarding; their
  output can contain sensitive payload data.
- Treat a recorded `complete` step as a claim to verify through safe
  read-back, not permission to rerun it. Do not change unrelated code or install
  unrelated SDK skills.

## Workflow

1. Read the approved plan, confirm the exact tenant and SANDBOX target, and
   select only incomplete `agent_vgs_cli` steps named in `selected_steps` with
   verified dependencies.
2. Inspect the installed CLI version and relevant help. Install or update the
   CLI yourself from the official getting-started documentation when needed.
3. Start `vgs login` yourself. Pause only for the user to finish browser
   authentication, then perform the least-privileged identity read.
4. Resolve organization and tenant identifiers from authenticated reads; never
   infer them from names or prefixes.
5. For each mutation, capture current state, prepare exact input, calculate its
   SHA-256 digest, and show a redacted preview containing the action, target,
   environment, input path, and digest.
6. Obtain explicit approval for that exact digest. Re-read target state before
   execution and reject stale approval.
7. Execute the exact approved command. For secret-producing operations, use the
   protected-output procedure in the execution contract.
8. Read back only allowlisted non-secret state and update the implementation-step status,
   `last_verified_at`, non-secret evidence, and execution log. Do not call a
   component complete without read-back evidence. Record remaining platform
   scopes when handing the plan onward, and do not mark the whole plan complete
   while any implementation step remains incomplete.

## Guarded documentation fallback

When a selected component skill is unavailable or current CLI help differs:

1. Search or browse only official `docs.verygoodsecurity.com` pages.
2. Compare the documented operation with the installed CLI's relevant nested
   `--help` output.
3. Continue only when both sources establish the exact CLI operation and the
   normal preview, approval, execution, and read-back contract can be applied.
4. If the CLI cannot perform the operation, record `blocked: unsupported by
   installed vgs-cli` and continue independent supported steps. Do not replace
   it with a user-run CLI command, guessed API request, or unapproved Dashboard
   mutation.
