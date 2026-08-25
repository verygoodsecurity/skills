---
name: vgs-show-js-guide
description: Integrate, implement, migrate, troubleshoot, or review VGS Show.js browser applications that securely reveal aliased text or images in hosted iframes.
metadata:
  author: verygoodsecurity
  version: '1.0.0'
---

# VGS Show.js Guide

Use this skill for web applications that load the versioned Show.js script and
use the global `VGSShow` API. Do not route native iOS, Android, or React Native
SDK work here.

## Resolve guidance

1. Read `references/AGENTS.md` before proposing or changing an integration.
2. Detect the installed Show.js version from the script URL in HTML, templates,
   CSP configuration, or generated page output. Do not infer it from unrelated
   npm dependencies: Show.js is loaded from the VGS CDN, not installed as an
   application package.
3. If the version is unknown, use the bundled snapshot and state that the
   application's loaded version could not be determined.
4. For exact signatures or newer behavior, consult the current official
   [Show.js documentation](https://docs.verygoodsecurity.com/vault/developer-tools/vgs-show/js)
   and its configuration and reference pages. Do not expose or depend on private
   repository content.

## Route the task

- `integrate`: verify the reveal route contract, pin a versioned CDN script with
  the documented integrity metadata, initialize `VGSShow`, request the minimum
  data, and render the secure frame.
- `implement`: add documented request, frame, event, copy, serializer, CNAME, or
  credential behavior in the customer's existing architecture.
- `migrate`: compare the loaded and target versions using official docs and
  changelog, update the script URL and integrity value together, and regression
  test reveal and error paths.
- `troubleshoot`: localize script loading, CSP/SRI, route, CORS/authentication,
  response selection, iframe rendering, and event failures before editing.
- `review`: check public API compatibility, iframe isolation, route scope,
  authentication, logging, copy behavior, cleanup, CSP/SRI, and tests.

Use synthetic SANDBOX aliases in examples and tests. Never place raw revealed
values, credentials, customer identifiers, or production configuration in code,
logs, analytics, fixtures, screenshots, or responses.

## Output contract

Begin with the detected script version and evidence, for example:

- `Detected Show.js 2.2.3 from the versioned script URL.`
- `Could not determine the loaded Show.js version; using bundled 2.2.3 guidance.`

Then distinguish verified application behavior from prerequisites that still
need route, browser, or authorization validation.
