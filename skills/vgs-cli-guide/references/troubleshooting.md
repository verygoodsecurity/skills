# Troubleshooting VGS CLI

Source: https://docs.verygoodsecurity.com/vault/developer-tools/vgs-cli/troubleshooting

## Debug mode

Have the customer re-run a failing command with `-d` / `--debug` in their own
terminal to get a stack trace for VGS Support:

```bash
vgs -d get routes --tenant <TENANT_ID>
```

Debug output can contain request context. Inspect and redact it before sharing
with VGS Support; never share tokens, client secrets, tenant credentials, or
private keys.

## Known issues (PyPI/Python distribution)

### "Authentication error occurred. Can't store password on keychain" during login

Sign the Python binary:

```bash
codesign -f -s - $(which python)
```

For a framework installation on macOS, the full application binary path may be
needed. Use the installed Python 3.11-or-newer version in this path:

```bash
codesign -f -s - /Library/Frameworks/Python.framework/Versions/<PYTHON_VERSION>/Resources/Python.app/Contents/MacOS/Python
```

### pip dependency conflicts on install

Install `vgs-cli` inside a virtualenv instead of the system Python.

### macOS Keychain access

- When macOS prompts for Keychain access during login, choose **Always
  Allow** for the VGS CLI.
- If access was accidentally denied and the prompt no longer appears: open
  the Keychain Access app, then Lock and Unlock the `login` keychain
  (`File → Lock/Unlock Keychain "login"`), and retry `vgs login`.

### Keyring error after a macOS update

`keyring.backends._OS_X_API.Error: (-25293, "Can't fetch password from
system")` — update the local Python to the latest version and reinstall
vgs-cli if needed.

## Session behavior

Interactive sessions expire after 30 minutes of inactivity; an authentication
failure mid-workflow usually just means `vgs login` again. Service-account
auth (env vars) does not have this problem.

## Support

For anything else: support@vgs.io.
