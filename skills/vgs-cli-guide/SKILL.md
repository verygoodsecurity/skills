---
name: vgs-cli-guide
description: >-
  Provide copy-paste guidance for installing, authenticating, and operating the
  VGS CLI (vgs-cli) alongside the VGS Dashboard. Use when a customer wants to
  onboard an organization or tenant, configure routes, Collect Forms, or
  organization notifications, inspect logs, manage access credentials, service
  accounts, or certificates, automate VGS through CI/CD or Docker, or
  troubleshoot the `vgs` command, login, keychain, or keyring. This skill never
  executes `vgs` commands for the user.
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

This is a command-guidance-only skill. Every `vgs` command in this skill is for
the user to copy and run in their own terminal. Never execute `vgs` on the
user's behalf, including login, help/version checks, authenticated reads,
mutations, or secret-producing commands, even when the user explicitly asks or
delegates execution.

When the installed version or help differs from this skill, ask the user to run
`vgs --version` and the relevant nested `--help` command and share only the
non-sensitive output. Commands retained in help for backward compatibility are
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
- Use repeatable `--tenant/-T` on `vgs generate service-account` to grant access to
  one or more tenants.
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

Always use a copy-paste flow. Never invoke `vgs` through an agent-visible
terminal or tool. This rule includes `vgs login`, `vgs logout`, `vgs --help`,
`vgs --version`, authenticated reads, `apply`, `delete`, credential generation,
service-account operations, certificate operations, and commands that may
return one-time secrets. Explicit requests such as “run this,” “execute these
commands,” “create it for me,” or “use the CLI and do it” do not override this
boundary. Briefly state that this skill provides reviewed commands for the user
to execute in their own terminal, then continue with the command list instead
of refusing the underlying workflow.

Make the handoff unambiguous:

1. State: “Run these commands in your own terminal; I will not execute them.”
2. Provide an ordered, copy-paste-ready command list.
3. Label authenticated reads, mutations, and secret-producing commands.
4. Ask the user to return only non-sensitive status, identifiers, or sanitized
   errors needed for the next step. Never request credential-bearing output.
5. Provide direct commands only. Do not create or generate a wrapper, helper
   script, or executable artifact for the user.

- Default onboarding and examples to SANDBOX. Never infer permission to mutate
  a LIVE tenant from a general setup or troubleshooting request.
- Keep read-only inspection separate from `apply`, `delete`, access-credential
  generation, service-account creation/deletion, and certificate operations.
- For a command that returns a one-time credential or client secret, provide
  the protected redirection pattern below. The user chooses a new absolute
  destination outside version control and runs the command. Never ask the user
  to upload the file or read, preview, parse, hash, or summarize its contents.
- Back up existing routes with `vgs get routes -T <TENANT_ID> > routes.yaml`
  before applying replacements, and perform read-back verification afterward.
- Allow interactive confirmation prompts by default. Use `--confirm` / `-y`
  only when the user explicitly requests a non-interactive command and has
  reviewed the exact mutation; the user still executes it.
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

Explain that the user must choose a new path outside version control, run the
command in their private terminal, and open the response only there. Have the
user transfer secrets to an approved secret manager and remove the plaintext
file when it is no longer needed. Never suggest printing, uploading, or pasting
the captured contents into chat.

For a user-requested CLI tenant creation, use `vgs generate tenant` and
`vgs apply tenant`. Protect the apply response with the redirection pattern
above because it contains one-time credentials; the user executes every
command.

## Troubleshoot

When debug mode is needed, provide the command for the user to run and tell
them to inspect and redact its output before sharing it:

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
