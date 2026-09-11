# Run VGS CLI in Docker

Use the official `verygoodsecurity/cli` image from Docker Hub. Always select a
published version explicitly: the `latest` tag is not updated by every VGS CLI
release and may expose an older command surface.

Sources:

- https://docs.verygoodsecurity.com/vault/developer-tools/vgs-cli/docker
- https://hub.docker.com/r/verygoodsecurity/cli/tags

## Select and verify a version

Choose a version from Docker Hub, then verify it before authentication:

```bash
export VGS_CLI_VERSION=<PUBLISHED_VERSION>
docker run --rm \
  verygoodsecurity/cli:${VGS_CLI_VERSION} --version
```

If the reported version is older than the skill or lacks an option shown in
this guide, inspect that image's generated `--help` instead of assuming the
newer contract is available.

## Docker Compose

Require `VGS_CLI_VERSION`; do not provide a fallback to `latest`:

```yaml
services:
  cli:
    image: verygoodsecurity/cli:${VGS_CLI_VERSION:?set VGS_CLI_VERSION}
    stdin_open: true
    tty: true
    entrypoint: bash
    ports:
      - "127.0.0.1:7745:7745"
      - "127.0.0.1:8390:8390"
      - "127.0.0.1:9056:9056"
```

Start and verify the service:

```bash
VGS_CLI_VERSION=<PUBLISHED_VERSION> docker compose up -d cli
docker compose exec cli vgs --version
docker compose exec cli vgs --help
```

## Interactive authentication

The container cannot open a browser on the host. Print the authorization URL
and open it manually:

```bash
docker compose exec cli vgs login --no-browser
```

Keep callback ports bound to loopback as shown above.

## Service-account authentication

For local Docker use, put the credentials in a gitignored `.env` file. For CI,
inject them from the CI platform's secret manager instead of writing a file:

```dotenv
VGS_CLI_VERSION=<PUBLISHED_VERSION>
VGS_CLIENT_ID=<SERVICE_ACCOUNT_CLIENT_ID>
VGS_CLIENT_SECRET=<SERVICE_ACCOUNT_CLIENT_SECRET>
```

Reference the environment file locally:

```yaml
services:
  cli:
    image: verygoodsecurity/cli:${VGS_CLI_VERSION:?set VGS_CLI_VERSION}
    stdin_open: true
    tty: true
    entrypoint: bash
    env_file:
      - .env
```

Never commit the populated file or print its values. Ensure the service account
has only the scopes and tenant grants needed by the command.

## Use and cleanup

Confirm the tenant and environment before authenticated operations:

```bash
docker compose exec cli vgs get routes --tenant <TENANT_ID>
docker compose down
```
