# VGS CLI Built-in Command Reference

Treat the installed command's generated help as the source of truth. This
reference matches the built-in command surface in the repository; installed
plugins may add commands.

Public documentation:
https://docs.verygoodsecurity.com/vault/developer-tools/vgs-cli/commands

## Contents

- [Explore the CLI](#explore-the-cli)
- [Authenticate](#authenticate)
- [Organizations and tenants](#organizations-and-tenants)
- [Organization notifications](#organization-notifications)
- [Service accounts](#service-accounts)
- [Access credentials](#access-credentials)
- [Routes](#routes)
- [Tenant resource documents](#tenant-resource-documents)
- [Collect Forms](#collect-forms)
- [Certificates](#certificates)
- [Logs](#logs)

## Explore the CLI

```bash
vgs --help
vgs --version
vgs <GROUP> --help
vgs <GROUP> <COMMAND> --help
```

Built-in root commands:

```text
apply
certificate
delete
generate
get
login
logout
logs
```

Use `-d` / `--debug` before a command to enable debug output. Inspect debug
output for secrets before sharing it.

## Authenticate

```bash
vgs login
vgs login --no-browser
vgs login --idp <IDP_ID>
vgs login --config <CONFIG_FILE>
vgs logout
```

`vgs login` options:

| Option | Purpose |
| --- | --- |
| `--browser` / `--no-browser` | Open the browser automatically or print the authorization URL. |
| `--idp <IDP_ID>` | Authenticate through a custom Identity Provider. |
| `--config <FILE>` | Read login defaults from a ConfigObj-format file. |

`VGS_CONFIG_FILE` overrides the default login configuration path; an explicit
`--config` path takes precedence. An explicit `--idp` takes precedence over
`[login] idp` in the selected file. When both `VGS_CLIENT_ID` and
`VGS_CLIENT_SECRET` are present, the CLI uses service-account authentication
automatically. Do not run interactive login in that mode.

## Organizations and tenants

In customer language, “vault” is the legacy name for a tenant. For
tenant-scoped commands, use the current `--tenant/-T` option and the exact
tenant identifier from Dashboard or the user. Do not infer validity from an
identifier prefix.

List the organizations visible to the authenticated identity:

```bash
vgs get organizations
```

List the tenants visible to the authenticated identity:

```bash
vgs get tenants
```

Create or select a tenant in Dashboard, then copy its exact identifier for
`--tenant/-T`. Confirm the tenant's environment before any mutation. Tenant
identifiers do not have a single standard prefix.

Use `get tenants` for tenant discovery and `generate tenant` / `apply tenant`
for CLI tenant creation. Canonical tenant creation uses a `kind: Tenant`
resource document.

## Organization notifications

Notification integrations are organization-scoped webhook configurations.
They require an active organization, organization-admin access, and a token
authorized for `notification-center:notifications:write`.

Read the provider catalog and current state:

```bash
vgs get notification-providers
vgs get notifications -O <ORGANIZATION_ID>
vgs get notification-events -O <ORGANIZATION_ID> <INTEGRATION_ID>
```

Apply configuration or change integration status:

```bash
vgs apply notification -O <ORGANIZATION_ID> -f notification.yaml
vgs apply notification -O <ORGANIZATION_ID> --enable <INTEGRATION_ID>
vgs apply notification -O <ORGANIZATION_ID> --disable <INTEGRATION_ID>
```

Delete after reviewing the exact organization and integration:

```bash
vgs delete notification -O <ORGANIZATION_ID> <INTEGRATION_ID>
```

`apply notification` requires exactly one of `--file/-f`,
`--enable`, or `--disable`. Deletion prompts unless `--confirm/-y` is supplied.
The configuration file uses `kind: NotificationIntegration`; omit `data.id` to
create and include it to update. Omitted events remain unchanged during an
update. Set an event's `enabled` field to `false` to disable it explicitly.

Provider-generated values such as the webhook signing secret must not appear
in the input file. The CLI masks them in read and apply output. Do not request
the raw secret or ask the customer to paste Dashboard notification details.
Read `workflows.md` for the backup, apply, and read-back sequence.

## Service accounts

Generate a template with one of the supported template names:

```bash
vgs generate service-account --template vgs-cli > service-account.yaml
vgs generate service-account --template calm \
  --tenant <TENANT_ID> > service-account.yaml
vgs generate service-account --template checkout \
  --tenant <TENANT_ID> \
  --var name=<SERVICE_ACCOUNT_NAME> > service-account.yaml
vgs generate service-account --template sub-account-checkout \
  --tenant <TENANT_ID> \
  --var sub_account_id=<SUB_ACCOUNT_ID> > service-account.yaml
vgs generate service-account --template payments-admin \
  --tenant <TENANT_ID> > service-account.yaml
```

Generate options:

| Option | Purpose |
| --- | --- |
| `--template, -t` | Required template: `vgs-cli`, `calm`, `checkout`, `sub-account-checkout`, or `payments-admin`. |
| `--var NAME=VALUE` | Supply a required template variable; repeat as needed. |
| `--tenant, -T <TENANT_ID>` | Grant access to a tenant; repeat for templates that allow multiple tenants. |

Inspect the generated YAML before applying it, then manage the account:

```bash
vgs apply service-account -O <ORGANIZATION_ID> -f service-account.yaml
vgs get service-accounts -O <ORGANIZATION_ID>
vgs delete service-account -O <ORGANIZATION_ID> <SERVICE_ACCOUNT_CLIENT_ID>
```

`apply` and `get` accept `-O/--organization`; `delete` requires it. The client
secret is returned only at creation. Provide the protected shell redirection
pattern from `SKILL.md`, using an absolute output path outside a Git worktree.
The customer runs it in their private terminal. Never execute the command or
display, request, or read the captured response. Read
`service-accounts.md` for scope and Dashboard guidance.

## Access credentials

```bash
vgs get access-credentials --tenant <TENANT_ID>
vgs generate access-credentials --tenant <TENANT_ID>
```

Both commands require `--tenant/-T`. Generation is a live mutation that returns
secret material. Provide generation with the protected shell redirection
pattern from `SKILL.md` and have the customer choose a new secure destination
and run it in their private terminal. Never execute the command or display,
request, or read the captured response.

## Routes

Generate starter resource documents:

```bash
vgs generate http-route
vgs generate mft-route
```

Read, apply, and delete routes:

```bash
vgs get routes --tenant <TENANT_ID> > routes.yaml
vgs get http-routes --tenant <TENANT_ID>
vgs apply routes --tenant <TENANT_ID> --filename routes.yaml
vgs delete routes --tenant <TENANT_ID> <ROUTE_ID>
```

| Command | Options and arguments |
| --- | --- |
| `get routes` | Required `--tenant/-T`. Returns a reusable top-level `data:` list. |
| `get http-routes` | Required `--tenant/-T`. Returns HTTP route resource documents. |
| `apply routes` | Required `--tenant/-T` and `--filename/-f`. Creates entries without an ID and updates entries with an ID. |
| `delete routes` | Required `--tenant/-T`, positional `ROUTE_ID`, optional `--confirm/-y`. |

Abbreviated reusable `get routes` payload:

```yaml
data:
- attributes:
    destination_override_endpoint: https://echo.apps.verygood.systems
    entries:
    - classifiers: {}
      config:
        condition: AND
        rules:
        - expression:
            field: PathInfo
            operator: matches
            type: string
            values: [/post]
        - expression:
            field: ContentType
            operator: equals
            type: string
            values: [application/json]
      id: 22222222-2222-2222-2222-222222222222
      operation: REDACT
      phase: REQUEST
      public_token_generator: UUID
      targets: [body]
      token_manager: PERSISTENT
      transformer: JSON_PATH
      transformer_config:
      - $.account_number
    host_endpoint: (.*).verygoodproxy.com
    id: 11111111-1111-1111-1111-111111111111
    port: 80
    protocol: http
    source_endpoint: '*'
  id: 11111111-1111-1111-1111-111111111111
  type: rule_chain
version: 1
```

The `generate http-route` resource envelope is not the top-level `data:` list
accepted by `apply routes`. For an existing tenant, start from `get routes`
output. Provide the backup, edit, apply, and read-back commands in order. The
customer reviews the exact mutation and runs every command.

## Tenant resource documents

```bash
vgs apply tenant-resources \
  --tenant <TENANT_ID> \
  --file resources.yaml \
  --dry-run true
```

`apply tenant-resources` requires `--tenant/-T` and `--file/-f`.
`--dry-run BOOLEAN` defaults to false. The file is multi-document YAML for
supported tenant resources. Use dry-run before a real apply, but do not treat it
as proof that remote authorization or provisioning will succeed.

## Collect Forms

```bash
vgs get forms --tenant <TENANT_ID>
vgs get form --tenant <TENANT_ID> <FORM_ID>
vgs apply form --tenant <TENANT_ID> --file form.yaml
vgs apply form --tenant <TENANT_ID> --json '<FORM_JSON>'
vgs delete form --tenant <TENANT_ID> <FORM_ID>
```

| Command | Options and arguments |
| --- | --- |
| `get forms` | Required `--tenant/-T`. |
| `get form` | Required `--tenant/-T` and positional `FORM_ID`; returns reusable Dashboard-compatible YAML. |
| `apply form` | Required `--tenant/-T`; exactly one of `--file/-f` or `--json`; optional `--confirm/-y` for replacement. |
| `delete form` | Required `--tenant/-T`, positional `FORM_ID`, optional `--confirm/-y`. |

Creating a new ID does not prompt. Replacing an existing form and deleting a
form prompt by default. Keep the prompt unless the user explicitly asks for a
non-interactive command after reviewing the exact mutation. The customer still
runs it. A 403 can indicate missing tenant permission or product entitlement;
do not work around it with browser headers or a pasted Dashboard token.

## Certificates

Every certificate subcommand requires `--tenant/-T`.

```bash
vgs certificate list -T <TENANT_ID>
vgs certificate create -T <TENANT_ID> \
  --domain example.com \
  --authority lets-encrypt
vgs certificate renew -T <TENANT_ID> --domain <CRT_ID_OR_DOMAIN>
vgs certificate get-public -T <TENANT_ID> --domain <CRT_ID_OR_DOMAIN>
vgs certificate download -T <TENANT_ID> \
  --domain <CRT_ID_OR_DOMAIN> \
  --output certificate.pem
vgs certificate delete -T <TENANT_ID> --domain <CRT_ID_OR_DOMAIN>
```

| Command | Options |
| --- | --- |
| `list` | Required `--tenant/-T`. |
| `create` | Required `--tenant/-T`, `--domain/-d`, and `--authority/-a` (`lets-encrypt` or `digicert`). |
| `renew` | Required `--tenant/-T` and `--domain/-d` accepting a certificate ID or domain. |
| `get-public` | Required `--tenant/-T` and `--domain/-d`. |
| `download` | Required `--tenant/-T`, `--domain/-d`, and `--output/-o/-f`. |
| `delete` | Required `--tenant/-T` and `--domain/-d`; optional `--confirm/-y`. Prefer a `CRT...` ID when a domain is ambiguous. |

Generate and upload Apple Pay certificate material:

```bash
vgs certificate generate-csr \
  -T <TENANT_ID> \
  --type apple-pay-merchant-identity \
  --name example-certificate \
  --output merchant-id.csr

vgs certificate generate-csr \
  -T <TENANT_ID> \
  --type apple-pay-payment-processing \
  --domain example.com \
  --output payment-processing.csr \
  --private-key-out payment-processing.key

vgs certificate upload \
  -T <TENANT_ID> \
  --cert-id CRT12345 \
  --file merchant-id.cer
```

`generate-csr` accepts:

- required `--tenant/-T`;
- required `--type` with `apple-pay-merchant-identity` or
  `apple-pay-payment-processing`;
- required `--name/-n/--domain/-d`;
- optional `--output/-o/-f`; and
- optional `--private-key-out`.

`upload` accepts required `--tenant/-T`, required
`--name/-n/--domain/-d/--cert-id`, and required `--file/-f`. Treat a generated
private key as secret material and never commit it. Provide
`--private-key-out` only after the customer chooses a secure local path that
will not enter agent output, logs, or version control. The customer runs it.

## Logs

Access logs:

```bash
vgs logs access --tenant <TENANT_ID>
vgs logs access -T <TENANT_ID> --since 1h
vgs logs access -T <TENANT_ID> --until 2020-08-01T12:30:45
vgs logs access -T <TENANT_ID> --tail 10 --output json
vgs logs access -T <TENANT_ID> --proxy http
```

| Option | Purpose |
| --- | --- |
| `--tenant, -T` | Required target tenant. |
| `--since` | Records newer than a duration (`30s`, `5m`, `3h`) or RFC 3339 timestamp. |
| `--until` | Records older than a duration or RFC 3339 timestamp. |
| `--tail` | Positive number of records from the end, or omit for all available records. |
| `--output, -o` | `yaml` (default) or `json`. |
| `--proxy, -P` | Filter by `http`, `sftp`, or `iso8583`. |

Access-log pages contain 30 records by default when `--tail` is unset or over
30; up to 1020 records can be fetched at a time.

Operation logs:

```bash
vgs logs operations \
  --tenant <TENANT_ID> \
  --request <REQUEST_ID> \
  --output yaml
```

`logs operations` requires `--tenant/-T` and `--request/-R`; output is
`yaml` by default or `json` with `--output/-o`.
