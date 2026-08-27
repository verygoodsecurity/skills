# Service Accounts

Source: https://docs.verygoodsecurity.com/vault/developer-tools/vgs-cli/service-account

A service account is a non-human client with limited access to an
organization's resources. Use one for vgs-cli scripts and CI/CD after the
interactive Dashboard and SANDBOX setup is working.

## Contents

- [Authentication](#authentication)
- [Creating via CLI](#creating-via-cli)
- [Naming](#naming)
- [Common documented scopes](#common-documented-scopes)
- [Deleting](#deleting)

## Authentication

Service accounts skip `vgs login` entirely. The CLI authenticates
automatically when both environment variables are present:

```bash
VGS_CLIENT_ID=<SERVICE_ACCOUNT_CLIENT_ID>
VGS_CLIENT_SECRET=<SERVICE_ACCOUNT_CLIENT_SECRET>
```

Inject these values from the automation platform's secret manager. Never ask a
user to paste them into chat, echo them, append them to a shell profile, or
store them in version control. A gitignored `.env` file is acceptable only for
controlled local use.

## Creating via CLI

1. Generate the general-purpose template and grant only the required tenants:

   ```bash
   vgs generate service-account \
     --template vgs-cli \
     --tenant <TENANT_ID> > service_account.yaml
   ```

   `--tenant/-T` is repeatable for templates that support multiple tenants.

   ```yaml
   apiVersion: 1.0.0
   kind: ServiceAccount
   data:
     # Max access-token lifetime in seconds (default 5 minutes)
     accessTokenLifespan: 300
     # Tenants the service account may access.
     # If none are listed, it has access to NO tenants.
     vaults:
       - <TENANT_ID>
     # Non-unique name, max 20 characters
     name: vgs-cli
     scopes:
       - name: access-logs:read
       - name: organizations:read
       - name: routes:write
       - name: vaults:write
   ```

   Other supported templates are `calm`, `checkout`, `sub-account-checkout`,
   and `payments-admin`. Some require `--var NAME=VALUE`; inspect
   `vgs generate service-account --help` and the generated YAML before use.

2. Edit `name`, `scopes`, and `vaults` to the minimum the automation needs.
   Do not add a scope merely because it appears in the table below.
3. Confirm the organization and SANDBOX/LIVE environment. Because apply returns
   a one-time secret, provide a command with protected shell redirection to a
   new absolute path outside a Git worktree. The customer runs it in their
   private terminal:

   ```bash
   (
     umask 077
     set -o noclobber
     vgs apply service-account \
       -O <ORGANIZATION_ID> \
       -f service_account.yaml \
       > /absolute/secure/new-service-account-response.yaml
   )
   ```

   Never execute this command. Do not ask the customer to upload, print, or
   paste the response, and do not read, preview, parse, or summarize it. Have
   the customer transfer the secret to an approved secret manager and return
   only a non-sensitive success status or sanitized error.

The output adds two fields:

```yaml
clientId: <SERVICE_ACCOUNT_CLIENT_ID>
clientSecret: <ONE_TIME_CLIENT_SECRET>
```

**The clientSecret is retrievable only at creation time. Transfer it directly
to the approved secret manager and do not retain the command output.**

List existing service accounts with:

```bash
vgs get service-accounts -O <ORGANIZATION_ID>
```

## Naming

`name` is max 20 characters and feeds the clientId pattern:
`<first 9 chars of ORGANIZATION_ID>-<name>-<5 random alphanumerics>`.

Each service account also gets a technical user with email
`<clientId>@vgs.dev`, visible under User Access Control in the VGS Dashboard
organization settings.

## Common documented scopes

The scopes available to an organization can change with enabled VGS products.
Use the generated template and current VGS documentation as the authority for
product-specific scopes; do not invent or broaden scopes.

| Scope | Description |
| --- | --- |
| `3ds:admin` | Invoke all 3DS endpoints (PayOpt) |
| `access-logs:read` | Read tenant access logs |
| `cards:read` | Read cards |
| `cards:write` | Write cards |
| `credentials:write` | Full management of vault credentials |
| `merchants:write` | Write merchants |
| `network-tokens:read` | Get network token status of an enrolled card |
| `network-tokens:write` | Enroll cards into network tokens, lifecycle actions |
| `notification-center:notifications:write` | Manage organization notification integrations when granted by VGS Identity |
| `organizations:read` | Read basic organization details |
| `preferences:write` | Read/create/update/delete tenant preferences |
| `routes:read` | Read all routes |
| `routes:write` | All routes operations |
| `rules:admin` | Manage all rules |
| `sub-accounts:admin` | Manage sub-accounts |
| `threeds:admin` | Configure 3DS providers |
| `transfers:admin` | Manage all transfers and reversals across accounts |
| `transfers:read` | Read transfers/reversals for a specific account |
| `transfers:write` | Create a transfer during a payment (Checkout) |
| `vaults:read` | Read tenant details (name, identifier) |
| `vaults:write` | Create and update tenants |

Specialized generated templates may contain additional product scopes, such as
financial instruments, gateways, or orders, that are not in this common table.

Scopes cannot be modified after creation; recreate the account to change them.
Limit: 50 service accounts per organization; contact
support@verygoodsecurity.com for more.

## Deleting

When the customer wants to delete a service account, provide this command for
them to run after they verify the exact organization and client ID:

```bash
vgs delete service-account -O <ORGANIZATION_ID> <SERVICE_ACCOUNT_CLIENT_ID>
```

Service accounts can also be created, viewed, and deleted in the VGS
Dashboard under Organization settings → Service Accounts.
