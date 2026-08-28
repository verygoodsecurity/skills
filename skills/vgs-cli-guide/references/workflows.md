# Customer Workflows

Use these workflows to connect an existing VGS Dashboard account to the CLI,
make controlled configuration changes, and hand the result to automation.

## Contents

- [Safety preflight](#safety-preflight)
- [Dashboard-assisted onboarding](#dashboard-assisted-onboarding)
- [Route change loop](#route-change-loop)
- [Organization notification loop](#organization-notification-loop)
- [Collect Form loop](#collect-form-loop)
- [CI/CD handoff](#cicd-handoff)
- [LIVE promotion](#live-promotion)

## Safety preflight

This skill always uses a copy-paste flow:

1. Never execute a `vgs` command for the customer, even when they explicitly
   ask the agent to run it.
2. Tell the customer that every command must be run in their own terminal.
3. Identify the VGS organization, exact tenant identifier, and SANDBOX or LIVE
   environment. Do not infer the identifier from a prefix.
4. Keep authenticated reads separate from mutations and label each command.
5. Protect secret-producing output with the redirection pattern in `SKILL.md`.
6. Ask for only non-sensitive status, identifiers, or sanitized errors. Do not
   request, print, retain, or paste tokens, client secrets, tenant credentials,
   or private keys.

Use SANDBOX for onboarding unless the user explicitly requests and approves a
LIVE operation.

## Dashboard-assisted onboarding

### 1. Establish Dashboard prerequisites

Have the customer sign in to the VGS Dashboard and select the intended
organization. Account creation, organization membership, product enablement,
entitlements, and user roles are Dashboard or VGS-managed prerequisites; do
not claim the CLI provisions them.

Ask whether the customer will use an existing SANDBOX tenant or wants to
create one in Dashboard. The customer performs Dashboard and CLI operations;
the skill provides instructions and commands only.

### 2. Install and authenticate interactively

Tell the customer to run these commands in their own terminal. Never run
`vgs login` for them:

```bash
python3 -m pip install vgs-cli
vgs --version
vgs login
```

Use `vgs login --no-browser` for headless access or `--idp <IDP_ID>` for a
custom Identity Provider. Do not use a service account for the initial
interactive setup unless the customer already provisioned one with the needed
organization and tenant access.

### 3. Reconcile Dashboard and CLI identity

Provide this authenticated read for the customer to run to confirm the
organization:

```bash
vgs get organizations
```

Match the Dashboard organization to the organization `AC...` ID. Select the
tenant in Dashboard, copy its exact tenant identifier, and verify that it is
SANDBOX or LIVE as intended. Tenant identifiers do not have a single standard
prefix; do not guess or rewrite the value before passing it to `--tenant`.

If the expected organization or tenant is absent, stop. Resolve Dashboard
membership or authorization instead of selecting a similarly named resource.

### 4. Create a SANDBOX tenant when requested

Have the customer create the tenant in Dashboard under the confirmed
organization, choose `SANDBOX`, and use the reviewed tenant name. Tenant
creation may expose credentials that must go directly to an approved secret
store; do not ask the customer to paste them into chat or an agent-visible
terminal.

After provisioning completes, have the customer copy the exact tenant
identifier from Dashboard and confirm the environment before composing any
tenant-scoped CLI command.

If the user explicitly requests tenant creation through the CLI, have them
generate and review the input locally:

```bash
vgs generate tenant > tenant.yaml
```

Provide apply with protected shell redirection because the response contains
one-time credentials. The customer chooses the destination and runs it in their
private terminal:

```bash
(
  umask 077
  set -o noclobber
  vgs apply tenant \
    -O <ORGANIZATION_ID> \
    -f tenant.yaml \
    > /absolute/secure/new-tenant-creation-response.yaml
)
```

The destination must be a new absolute path outside a Git worktree. Never run
the command or ask the customer to upload, print, or paste the response. Have
the customer open it only in their private terminal, store the credentials in
an approved secret manager, and return only non-sensitive status and the exact
tenant identifier needed for subsequent CLI commands.

### 5. Establish a read-only baseline

```bash
vgs get routes -T <TENANT_ID> > routes.baseline.yaml
vgs get forms -T <TENANT_ID>
vgs certificate list -T <TENANT_ID>
```

Have the customer run only the reads relevant to their integration. A 403 is an
authorization or entitlement blocker, not a reason to copy browser headers or
request a pasted Dashboard token.

### 6. Configure the requested integration

Choose the narrow workflow:

- For proxy routing, use the route change loop below.
- For webhook notifications, use the organization notification loop below.
- For a Collect Form, use the Collect Form loop below.
- For certificates, inspect `commands.md` and confirm the certificate type and
  private-key handling before providing commands.
- For CI/CD, complete the interactive configuration first, then use the CI/CD
  handoff below.

### 7. Verify in CLI and Dashboard

Have the customer read the changed resource back through the CLI, refresh the
matching tenant in the Dashboard, and compare identifiers and relevant
configuration. When the integration can make a safe synthetic request, provide
the relevant request and log commands for the customer to run:

```bash
vgs logs access -T <TENANT_ID> --tail 10
vgs logs operations -T <TENANT_ID> -R <REQUEST_ID>
```

Report provisioning, configuration read-back, application integration, and
end-to-end request validation as separate outcomes.

## Route change loop

1. Confirm the tenant and environment.
2. Back up current state:

   ```bash
   vgs get routes -T <TENANT_ID> > routes.before.yaml
   cp routes.before.yaml routes.proposed.yaml
   ```

3. Edit `routes.proposed.yaml`; preserve the top-level `data:` list.
4. Review the semantic diff and the request/response phases, operations,
   classifiers, targets, transformers, and destinations.
5. Provide this mutation for the customer to run after reviewing the exact
   tenant, environment, and semantic diff:

   ```bash
   vgs apply routes -T <TENANT_ID> -f routes.proposed.yaml
   ```

6. Read back and compare:

   ```bash
   vgs get routes -T <TENANT_ID> > routes.after.yaml
   diff -u routes.proposed.yaml routes.after.yaml
   ```

7. Have the customer verify the same routes in Dashboard and, when appropriate,
   run a synthetic request and inspect logs.

Do not assume `apply routes` replaces the whole collection atomically: entries
with IDs are updated and entries without IDs are created. Delete a route only
through the explicit `delete routes` command and its confirmation.

## Organization notification loop

1. Confirm the active organization and organization-admin access. A 403 is an
   authorization blocker; do not work around it with copied Dashboard headers
   or tokens.
2. Have the customer run these reads in their own terminal:

   ```bash
   vgs get notification-providers
   vgs get notifications -O <ORGANIZATION_ID> \
     > notifications.before.yaml
   ```

3. Prepare `notification.proposed.yaml`. The customer must not add a webhook
   signing secret; VGS generates it and the CLI masks it:

   ```yaml
   apiVersion: 1.0.0
   kind: NotificationIntegration
   data:
     provider: webhook
     name: Route alerts
     description: Route lifecycle notifications
     providerParameters:
       - name: url
         value: https://example.com/webhooks/vgs
     events:
       - id: route.updated
         enabled: true
         resourceIds:
           - <TENANT_ID>
   ```

   Omit `data.id` to create. Include the exact integration ID to update.
   Omitted events remain unchanged; use `enabled: false` for an intentional
   event disable.
4. After reviewing the organization, destination URL, events, and source IDs,
   have the customer run the mutation:

   ```bash
   vgs apply notification \
     -O <ORGANIZATION_ID> \
     -f notification.proposed.yaml
   ```

5. Read back the integration and configured events:

   ```bash
   vgs get notifications -O <ORGANIZATION_ID> \
     > notifications.after.yaml
   vgs get notification-events \
     -O <ORGANIZATION_ID> \
     <INTEGRATION_ID>
   ```

6. Verify the same integration in Dashboard and send only a safe synthetic
   event to the reviewed destination. Treat CLI read-back and webhook delivery
   as separate outcomes.

Enable or disable an existing integration without replacing its metadata:

```bash
vgs apply notification -O <ORGANIZATION_ID> \
  --enable <INTEGRATION_ID>
vgs apply notification -O <ORGANIZATION_ID> \
  --disable <INTEGRATION_ID>
```

Provide deletion only after the customer confirms the exact integration. The
customer runs it and then reads the list back:

```bash
vgs delete notification -O <ORGANIZATION_ID> <INTEGRATION_ID>
vgs get notifications -O <ORGANIZATION_ID>
```

## Collect Form loop

List forms and retrieve an existing form as reusable Dashboard-compatible
YAML:

```bash
vgs get forms -T <TENANT_ID>
vgs get form -T <TENANT_ID> <FORM_ID> > form.before.yaml
```

Prepare exactly one input form:

```bash
vgs apply form -T <TENANT_ID> -f form.proposed.yaml
# or
vgs apply form -T <TENANT_ID> --json '<FORM_JSON>'
```

Before providing apply, inspect the form ID, name, and configuration. The
customer runs the command. Creating a new ID does not prompt; replacing an
existing ID does. Keep the replacement prompt unless the customer explicitly
asks for a non-interactive command after reviewing the exact replacement.

After apply:

```bash
vgs get form -T <TENANT_ID> <FORM_ID> > form.after.yaml
```

Compare the result with the proposed input and verify the same form in the
Dashboard. Treat a successful form configuration read-back separately from
frontend integration, visual behavior, accessibility, and end-to-end
tokenization; those require their own evidence.

Provide deletion only after the customer verifies the exact target. The
customer runs it:

```bash
vgs delete form -T <TENANT_ID> <FORM_ID>
```

## CI/CD handoff

Have the customer create a service account either in Dashboard under
Organization settings or through the CLI. The built-in CLI templates are for
payment-credential workflows. For a read-only integration that must not return
PCI-sensitive PAN or CVC fields, use the no-PCI template:

```bash
vgs generate service-account \
  --template read-credentials-no-pci \
  --tenant <TENANT_ID> \
  --var name=<SERVICE_ACCOUNT_NAME> > service-account.yaml
```

Use `public-credential-collect` for card, network-token, and 3DS writes. Use
`read-credentials-with-pci` only for a PCI-compliant client that must retrieve
PAN or CVC data. Use `credentials-admin` only when the integration needs the
complete read/write scope set. For unrelated automation, create a separately
reviewed least-privilege service account instead of broadening one of these
templates.

Review the generated YAML. Because apply returns a one-time client secret,
provide protected shell redirection to a new absolute destination outside a
Git worktree. The customer runs it in their private terminal:

```bash
(
  umask 077
  set -o noclobber
  vgs apply service-account \
    -O <ORGANIZATION_ID> \
    -f service-account.yaml \
    > /absolute/secure/new-service-account-response.yaml
)
```

Never execute this command or ask the customer to upload, print, or paste the
response. Have them transfer the secret to an approved secret manager and
return only a non-sensitive success status or sanitized error.

Store the one-time `clientId` and `clientSecret` directly in the CI platform's
secret manager as `VGS_CLIENT_ID` and `VGS_CLIENT_SECRET`. Do not commit a
populated `.env` file or append the secret to a shell profile.

Validate the identity with the least-privileged read required by the pipeline.
Then keep the pipeline sequence explicit:

1. fetch and preserve the current configuration;
2. validate or review the proposed YAML;
3. require the deployment environment's approval gate;
4. apply to the exact tenant;
5. read back and compare; and
6. retain non-secret audit evidence.

## LIVE promotion

Do not treat a successful SANDBOX setup as authorization to mutate LIVE.

Before promotion:

1. identify the separate LIVE tenant identifier and organization;
2. fetch its current configuration rather than assuming SANDBOX parity;
3. remove environment-specific endpoints, IDs, credentials, and test values;
4. review product enablement and service-account scopes for LIVE;
5. have the customer explicitly approve the exact LIVE commands before they run
   them; and
6. prepare read-back and rollback procedures.

Have the customer run each LIVE mutation separately and verify it before
continuing. Report configuration deployment, Dashboard visibility, and
production traffic validation independently.
