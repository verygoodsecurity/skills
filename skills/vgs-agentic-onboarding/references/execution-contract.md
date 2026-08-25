# Execution contract

## Authorization states

Planning approval authorizes preparation, CLI installation, authentication, and
read-only discovery. Each mutation still needs approval of its exact preview
digest. An approval expires when the target, input, command, environment, or
current remote state changes.

## Preview record

Record these non-secret fields in `VGS_ONBOARDING.md` before requesting
approval:

- operation and exact target identifier;
- confirmed organization, tenant, and `SANDBOX` environment;
- exact command with credentials and secret-bearing values omitted;
- absolute input path and SHA-256 digest;
- current-state digest or explicit `absent` state;
- expected read-back fields and failure boundary.

Do not show full customer route documents when a summary of protocol, direction,
destination hostname, filter phase, operation, and selector is sufficient.

## Secret-producing commands

Tenant and service-account creation can return one-time credentials. Before
execution, ask the user to approve a new absolute protected output path outside
the repository. The agent runs the command with restrictive permissions and
redirects both stdout and stderr to that path. The terminal tool may expose
only the process exit status; the agent must never open, print, summarize, hash,
or otherwise read the protected file.

Use the shell's `umask 077` and `noclobber` protections. Never use a repository
path, an unresolved environment variable, `$HOME`, or `~`. If the file already
exists, stop rather than overwrite it. The user moves the secret into an
approved secret manager and deletes the plaintext when no longer needed.

If a secret-producing command fails, do not read its captured output. Report
only the exit status and direct the user to inspect the protected file locally.

## Safe read-back

Allowlist identifiers, resource type, environment, protocol, route direction,
source and destination endpoints, ports, HTTP method and proxy path, content
type, filter conditions, phase, operation, selectors, alias format, storage,
status, timestamps, and value-free synthetic schema shapes containing only
field names and types. Extract only these fields at the command boundary so a
full response does not enter the conversation. Do not read service-account
secrets, access credentials, raw values, real aliases paired with raw values,
traffic payloads, or debug logs.

For route changes, save the full pre-change response outside the conversation,
apply only the approved resource, fetch state again, and compare the allowlisted
structure. Do not claim that a route works until synthetic SANDBOX traffic
validates the intended redaction or reveal behavior.

Secret-dependent application validation may use only an approved secret
manager or injection mechanism that does not expose the credential to the
agent, command text, terminal output, repository, or plan. If protected
injection is unavailable, leave validation blocked and record the missing
external prerequisite; never ask the user to paste or type a secret.
