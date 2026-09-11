---
name: vgs-cli-guide
description: >-
  Provide copy-paste guidance for installing, authenticating, and operating the
  VGS CLI (vgs-cli) alongside the VGS Dashboard. Use when a customer wants to
  onboard an organization or tenant, configure routes, Collect Forms, or
  organization notifications, inspect logs, manage access credentials, service
  accounts, or certificates, automate VGS through CI/CD or Docker, or
  troubleshoot the `vgs` command, login, keychain, or keyring. Provide reviewed
  commands by default, or execute them when the user explicitly delegates CLI
  operation with the required environment and target authorization.
license: MIT
metadata:
  author: Very Good Security
  source: https://docs.verygoodsecurity.com/vault/developer-tools/vgs-cli
---

# VGS CLI Guide

Use the VGS CLI (`vgs`) to manage VGS tenants, routes, Collect Forms,
organization notifications, access credentials, service accounts,
certificates, and logs from a terminal or an automation pipeline. Treat the
current installed CLI help as the exact command contract; installed plugins
may add commands that are not part of the built-in surface documented by this
skill.

Provide copy-paste commands by default. When the user explicitly delegates CLI
execution, the agent may run `vgs` within the authorized scope. Before a live
authenticated read or mutation, confirm the applicable organization, exact
tenant, SANDBOX or LIVE environment, and requested operation. Delegation to use
the CLI does not authorize broader discovery, additional mutations, or a LIVE
operation that the user did not specify.

When the installed version or help differs from this skill, run `vgs --version`
and the relevant nested `--help` command when execution was delegated;
otherwise ask the user to run them and share only non-sensitive output.
Commands retained in help for backward compatibility are
not the recommended workflow; follow this skill's deprecation guidance even
when an old command remains visible. For a skill installed through skills.sh,
have the user update it with:

```bash
npx skills update vgs-cli-guide
```

Point the user to the public VGS documentation or support@vgs.io when current
help and the bundled references do not establish an answer. Do not guess.

## Install and verify

```bash
python3 -m pip install vgs-cli   # requires Python 3.11+
vgs --version
vgs --help
```

Use a virtual environment if pip reports dependency conflicts. For Docker or
containerized CI, read `references/docker.md` and require an explicit image
version instead of relying on the stale `latest` tag.

## Authenticate

Choose authentication based on whether a human is present.

For an interactive personal account:

```bash
vgs login
vgs login --no-browser
vgs login --idp <IDP_ID>
```

Login uses browser OAuth and may ask permission to use the OS key management
system. Allow it so the CLI can store the encryption key used for local token
files. Interactive sessions expire after 30 minutes of inactivity; use
`vgs logout` to end one manually.

For scripts and CI/CD, use a least-privilege service account through
`VGS_CLIENT_ID` and `VGS_CLIENT_SECRET`; do not run `vgs login` in automation.
Read `references/service-accounts.md` before creating or using one. Never ask a
user to paste a client secret or token into chat, and never print, log, or
commit one.

## Select the workflow

Read `references/workflows.md` when the user wants to:

- onboard from the VGS Dashboard to the CLI;
- create or select a SANDBOX tenant in Dashboard and establish a CLI baseline;
- change routes with backup and read-back verification;
- create or replace a Collect Form and verify Dashboard parity;
- hand an interactive setup off to CI/CD with a service account; or
- promote reviewed configuration toward a LIVE tenant.

For a first-time setup, start with the Dashboard-assisted onboarding flow.
Keep Dashboard-only account, organization, entitlement, and product-enablement
steps distinct from operations the CLI can perform.

## Terminology: vault and tenant

When a customer says “vault” in general conversation, interpret it as the
legacy name for a VGS tenant unless the context clearly refers to a CLI
command, resource kind, or API object. Translate the request into the current
tenant terminology and use the exact tenant identifier from Dashboard or the
user with `--tenant/-T`. Do not infer a tenant identifier from its prefix.

Use `vgs get tenants`, `vgs generate tenant`, and `vgs apply tenant` for tenant
discovery and creation. Canonical tenant creation uses `kind: Tenant`.

Keep “vault” literal only when:

- the user explicitly asks to understand or migrate a deprecated command; or
- a legacy API object, payload field, documentation URL, or Dashboard label
  uses that spelling.

Use `--tenant/-T` with `vgs generate service-account`. Do not guess whether an
identifier is valid from a prefix; use the exact tenant identifier shown in
Dashboard or explicitly supplied by the user.

## Core command pattern

Use `get` for reading tenant-scoped state, `generate -> edit -> apply` for
supported YAML resources, and `delete` for removal. Tenant creation and
selection happen in Dashboard:

```bash
# Read state
vgs get organizations
vgs get tenants
vgs get routes --tenant <TENANT_ID>
vgs get notifications --organization <ORGANIZATION_ID>

# Generate a starter route resource document
vgs generate route --protocol http
vgs generate route --protocol sftp

# Create or update a tenant-scoped resource from YAML
vgs apply routes --tenant <TENANT_ID> -f routes.yaml

# Customer-run removal after reviewing the exact target
vgs delete routes --tenant <TENANT_ID> <ROUTE_ID>
vgs delete notification --organization <ORGANIZATION_ID> <INTEGRATION_ID>
```

Apply these conventions precisely:

- Use `--tenant` / `-T` with the exact tenant identifier from Dashboard or the
  user; tenant identifiers do not have a single standard prefix.
- Use `--organization` / `-O` for an organization ID such as `AC...`.
- The `vgs-cli` service-account template accepts zero or more `--tenant/-T`
  options; payment-credential templates require exactly one.
- Use `-f` for an input file where help exposes it and `-o` for an output file.
- Use `vgs --help`, `vgs <GROUP> --help`, and
  `vgs <GROUP> <COMMAND> --help` before relying on an unfamiliar option.

Read `references/commands.md` before composing a non-trivial command. It maps
every built-in command and its options, including service-account templates,
route payloads, organization notifications, Collect Forms, certificates, and
logs.

## Configuration

Store login defaults in the ConfigObj-format file `~/.vgs/config` on
Linux/macOS or `C:\Users\USERNAME\.vgs\config` on Windows:

```ini
[login]
idp = AC1234567890123456789012
```

Only `vgs login` exposes the configuration option:

```bash
vgs login --config ./config
```

Use `VGS_CONFIG_FILE` to override the login configuration path. For supported
login values, an explicit `--idp` takes precedence over `[login] idp` from the
selected configuration file; an explicit `--config` takes precedence over the
`VGS_CONFIG_FILE` path.

## Safety gates

Support both guidance and delegated execution:

1. Without explicit delegation, provide an ordered, copy-paste-ready command
   list for the user to run.
2. With explicit delegation, execute only the requested CLI workflow and stay
   within the confirmed organization, tenant, environment, and operation.
3. Label authenticated reads, mutations, and secret-producing commands before
   asking for authorization or executing them.
4. Return only non-sensitive status, identifiers, and sanitized errors. Never
   place credential-bearing output in chat or agent-visible tool output.

- Default onboarding and examples to SANDBOX. Never infer permission to mutate
  a LIVE tenant from a general setup or troubleshooting request.
- Keep read-only inspection separate from `apply`, `delete`, access-credential
  generation, service-account creation/deletion, and certificate operations.
- For a command that returns a one-time credential or client secret, provide
  the protected redirection pattern below. The user chooses or approves a new
  absolute destination outside version control. If execution is delegated, run
  the command only with stdout redirected directly to that destination; never
  capture the response in agent-visible output or read, preview, parse, hash,
  or summarize it.
- Back up existing routes with `vgs get routes -T <TENANT_ID> > routes.yaml`
  before applying replacements, and perform read-back verification afterward.
- Allow interactive confirmation prompts by default. Use `--confirm` / `-y`
  only when the user explicitly requests a non-interactive command and has
  reviewed the exact mutation.
- Treat tenant credentials, service-account secrets, access tokens, and
  certificate private keys as secrets. Use a secret manager for automation;
  never put them in shell history, logs, chat, or version control.
- Scope service accounts to the minimum required scopes and tenants. No listed
  tenants means no tenant access.

Wrap every secret-producing command in a standard shell subshell that uses a
restrictive umask and refuses to overwrite an existing file:

```bash
(
  umask 077
  set -o noclobber
  vgs <SECRET_PRODUCING_COMMAND> > /absolute/secure/new-response.yaml
)
```

The user must choose or approve a new path outside version control and open the
response only in their private environment. When execution is delegated,
verify that the destination is outside version control and does not exist, but
do not inspect the response contents. Have the user transfer secrets to an
approved secret manager and remove the plaintext file when it is no longer
needed. Never suggest printing, uploading, or pasting the captured contents
into chat.

For a user-requested CLI tenant creation, use `vgs generate tenant` and
`vgs apply tenant`. SANDBOX and LIVE both use bounded provisioning, create the
payment account, apply environment-specific account configuration, and keep
private recovery state. SANDBOX additionally performs merchant setup and
creates a CLI-managed `default` Collect Form with card brand and card type
enabled. Dashboard tenant creation does not yet create this form, and LIVE
creation never creates it. Protect the apply response with the redirection
pattern above because it contains one-time credentials. If either environment
reports partial completion, run or provide the same `vgs apply tenant --file`
command again. The CLI automatically
continues the existing operation. When the matching operation is already
complete, the CLI verifies the tenant and reports that no changes were made.
To create a separate tenant, update the file to use a unique tenant name.
Recovery files are managed internally. When output is protected with
`noclobber`, the repeated command needs a new secure response destination.

## Troubleshoot

When debug mode is needed, keep its potentially sensitive output out of
agent-visible tools. Without a safe redirected destination, have the user run
it and inspect and redact its output before sharing it:

```bash
vgs -d get routes --tenant <TENANT_ID>
```

Read `references/troubleshooting.md` before proposing keychain, keyring,
codesign, or reinstall steps.

## Reference files

- `references/workflows.md` - Dashboard-assisted onboarding, route and Collect
  Form change loops, CI/CD handoff, and LIVE promotion gates.
- `references/commands.md` - built-in command and option reference with exact
  tenant terminology and payload examples.
- `references/service-accounts.md` - templates, scopes, Dashboard creation,
  deletion, tenant restrictions, and secret-safe automation.
- `references/docker.md` - version-pinned Docker Compose usage and both
  authentication modes.
- `references/troubleshooting.md` - known installation, login, keychain, and
  keyring issues.
