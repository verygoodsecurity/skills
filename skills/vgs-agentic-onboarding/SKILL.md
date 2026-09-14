---
name: vgs-agentic-onboarding
description: Execute an approved VGS SANDBOX onboarding plan with vgs-cli, including tenant, route, and service-account setup, when the user wants the agent—not the user—to run CLI commands.
---

# VGS agentic onboarding

Use this skill only after needs discovery produced a confirmed
`VGS_ONBOARDING.md` and the user explicitly authorized the selected
implementation scope. This is the execution skill for agent-run VGS CLI
onboarding; do not route this workflow through a command-guidance-only skill
that requires user copy/paste. The onboarding file describes architecture and
scope. It does not itself authorize VGS resource mutations.

The plan may be shared across Dashboard, backend, frontend, and mobile
implementations without assuming company roles. Before installing the CLI or
loading component skills, inspect `requested_platforms`,
`detected_project_scopes`, `selected_steps`, and `implementation_steps`.
Continue only when an incomplete `agent_vgs_cli` step is selected and its
dependencies have current non-secret evidence. Dashboard-only or
application-only selection does not authorize CLI execution.

Read [references/execution-contract.md](references/execution-contract.md)
before any authenticated read or mutation. Choose the execution path before
resolving a tenant:

- **Structured setup path:** Use only when installed `vgs --help` confirms
  `vgs generate setup`, `vgs setup plan`, and `vgs setup apply`. It creates a
  versioned `VGS-SETUP.yaml`, saves an immutable plan, and applies the approved
  saved plan.
- **Legacy path:** Applies to Dashboard changes, application code, and CLI work
  that the installed setup workflow does not support. It retains the
  per-operation approval contract in the execution reference.

For an existing tenant in either path, require a non-null exact `tenant_id` and
`environment_confirmed: true` before mutation. Verify through safe reads that
the authenticated tenant is that identifier and is SANDBOX; block on any
mismatch and never infer an identifier from a name or prefix.

For a new tenant, use only the structured setup path. `tenant_id` may be null
until creation succeeds. Before authenticated planning, require the approved
SANDBOX organization, proposed tenant name, creation intent, and selected
resources in `VGS_ONBOARDING.md` and `VGS-SETUP.yaml`. Verify the authenticated
organization through a safe identity read. After apply, record only the
non-secret tenant identifier returned by read-back; require that exact
identifier for later work outside the saved plan.

Discover the current public Skills catalog before resolving dependencies. For
each selected component or SDK skill, read its catalog entry and use the exact
published install source and command declared there. Cross-product onboarding
and CLI skills may be installed from `verygoodsecurity/skills`; SDK skills may
declare their product repository instead. Never replace a catalog-declared
product source with the catalog repository merely because the skill is listed
there.

Install only selected skills at global scope so the customer repository remains
unchanged. Use the catalog-declared source in the normal install form:

```bash
npx skills add <CATALOG_DECLARED_SOURCE> --global --skill <COMPONENT_SKILL> --yes
```

Confirm each selected global skill is available before continuing. Never use a
project-local installation, ask the user to run the command, or leave `.agents/`
or `skills-lock.json` in the customer repository. Use the guarded fallback when
the catalog-declared installation fails, would modify the project, or the
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

1. Read the approved scope and select only incomplete `agent_vgs_cli` steps
   named in `selected_steps` with verified dependencies. Identify whether the
   work is structured setup, legacy work, or both.
2. Inspect the installed CLI version and relevant help. Install or update the
   CLI yourself from the official getting-started documentation when needed.
3. Start `vgs login` yourself. Pause only for the user to finish browser
   authentication, then perform the least-privileged identity read.
4. For structured setup, generate `VGS-SETUP.yaml` from the installed CLI
   template, populate only the confirmed selected resources, and run `vgs setup
   plan -f VGS-SETUP.yaml`. Planning must not mutate. Present the resulting
   redacted preview, saved-plan location, and SHA-256 digest.
5. Obtain one explicit customer approval for that exact saved-plan digest. Do
   not treat an `approved` field in either artifact as consent. The agent records
   the customer decision in `VGS_ONBOARDING.md`; the CLI verifies the saved-plan
   integrity and remote-state preconditions.
6. Apply the complete saved plan with `vgs setup apply --plan <SAVED_PLAN>`.
   Do not add a CLI `--approval` flag. Use `--operation <OPERATION_ID>` only when
   the customer selected that already-previewed operation and every dependency
   is verified complete in the same saved plan; never use it to broaden scope or
   bypass the whole-plan approval.
7. For new-tenant apply, use the protected-output procedure and the installed
   CLI's required credentials-output argument. For every structured operation,
   read back allowlisted non-secret state and update the implementation-step
   status, `last_verified_at`, non-secret evidence, and execution log.
8. For legacy work, follow the per-operation preview, exact-target,
   digest-bound approval, drift check, execution, and read-back procedure in
   the execution contract. A saved setup plan never authorizes Dashboard or
   application changes.
9. Do not call a component complete without read-back evidence. Record
   remaining platform scopes when handing the plan onward, and do not mark the
   whole plan complete while any implementation step remains incomplete.

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
