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

1. Choose the least-privilege payment-credential template and grant it access
   to exactly one tenant. This example reads cards without returning
   PCI-sensitive PAN and CVC fields:

   ```bash
   vgs generate service-account \
     --template read-credentials-no-pci \
     --tenant <TENANT_ID> \
     --var name=<SERVICE_ACCOUNT_NAME> > service_account.yaml
   ```

   `--var name=...` is required and must contain 1–20 characters. Each
   built-in template requires exactly one `--tenant/-T`.

   ```yaml
   apiVersion: 1.0.0
   kind: ServiceAccount
   data:
     name: <SERVICE_ACCOUNT_NAME>
     vaults:
       - <TENANT_ID>
     scopes:
       - name: cards:read
       - name: card-attributes:read
       - name: network-tokens:read
       - name: account-validations:write
       - name: account-validations:read
       - name: account-reference-numbers:read
   ```

   Use `public-credential-collect` to write cards, network tokens, and 3DS data.
   Use `read-credentials-with-pci` only for a PCI-compliant client that must
   retrieve PAN or CVC data. Use `credentials-admin` only when an integration
   needs the complete read/write scope set. These templates are specific to
   payment-credential workflows; create a separately reviewed least-privilege
   configuration for unrelated automation.

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
| `3ds:read` | Read 3DS authentication status |
| `3ds:write` | Initialize and authenticate 3DS transactions |
| `access-logs:read` | Read tenant access logs |
| `account-reference-numbers:read` | Read account reference numbers |
| `account-validations:read` | Read account validation results |
| `account-validations:write` | Create account validation requests |
| `card-attributes:read` | Read card attributes |
| `cards:read` | Read cards |
| `cards:read-pci` | Read PCI-sensitive PAN and CVC fields for PCI-compliant clients |
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

Generated templates may contain product scopes that are not available until
the corresponding VGS product is enabled for the organization.

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
