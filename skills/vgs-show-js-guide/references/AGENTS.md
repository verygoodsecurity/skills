# VGS Show.js integration policy

**SDK Version: 2.2.3**

This is the public-safe policy for integrating Show.js into customer web
applications. The official documentation is the source of truth for published
CDN versions, integrity hashes, and supported public APIs:
https://docs.verygoodsecurity.com/vault/developer-tools/vgs-show/js

## Security boundary

Show.js reveals data inside a VGS-hosted iframe so the application and third-
party page scripts do not receive the raw value. Preserve that boundary:

- Keep raw values out of application state, DOM outside the secure iframe,
  callbacks, logs, analytics, error reports, tests, and screenshots.
- Send only aliases and the minimum request data needed by a least-privilege
  reveal route. The route must reveal on the response path and authorize the
  end user independently of the alias.
- Treat `requestSuccess` as HTTP completion and `revealSuccess` as successful
  selection/rendering. Test and handle both failure events.
- Use copy-to-clipboard only when the product explicitly requires it. The copy
  control must be created with `show.copyFrom()` so the value stays within the
  secure-frame flow.
- Use synthetic aliases and SANDBOX configuration for examples and development.

## Script loading

Load an explicit CDN version rather than an unversioned or floating URL. Use the
exact `integrity` value published for that version and include
`crossorigin="anonymous"`; never invent, reuse across versions, or omit the hash
when the official configuration page provides one. Keep Content Security Policy
allowlists limited to the documented Show.js script, frame, and connection
origins.

Show.js exposes `window.VGSShow`; it is not an npm dependency. Ensure integration
code runs after the script has loaded and surfaces a safe fallback if loading is
blocked by CSP, SRI, the network, or an extension.

## Core flow

1. Confirm an inbound reveal route exists and record its method, path,
   authentication mechanism, payload, and response shape without secrets.
2. Create the instance with `VGSShow.create(vaultId, stateCallback)` and call
   `.setEnvironment(environment)` before creating frames.
3. Call `show.request()` with a unique `name`, route `method` and `path`, the
   minimum alias-bearing `payload`, and a `jsonPathSelector` matching the
   proxied response. `jsonPathSelector` is not needed for blob/image responses.
4. Register frame events before rendering, then call `frame.render(selector,
   styles)` on an existing, dedicated wrapper element.
5. On teardown, unregister application listeners and call `frame.unmount()`.

Supported environment shapes are `sandbox`, `live`, and documented regional
variants. Keep vault, environment, route ID, and custom hostname in trusted
deployment configuration rather than accepting them from end-user input.

## Public API

### Instance

- `VGSShow.create(vaultId, stateCallback?)`
- `show.setEnvironment(environment)`
- `show.setCname(hostname)` for a Dashboard-activated custom hostname
- `show.setRouteId(routeId)` when a specific reveal route must be selected
- `show.request(params)` returns a secure frame
- `show.copyFrom(textFrame, params, callback?)` returns a secure copy-button
  frame; create the target frame first
- `show.SERIALIZERS.replace(oldValue, newValue, count?)` returns a formatting
  serializer for request or copy configuration

### Request parameters

Required: `name`, `method`, and `path`.

Common optional parameters: `payload`, `headers`, `jsonPathSelector`,
`htmlWrapper` (`text` or `image`), `serializers`, `displayBase64Image`,
`decodeDataFrom`, `xhrResponseType`, and `withCredentials`.

Prefer `htmlWrapper` over deprecated `responseDataType`, and
`displayBase64Image` over deprecated `decodeFromBase64`. Use only methods and
parameter combinations supported by the official reference for the loaded
version.

### Frame lifecycle

- `frame.render(selector, styles?)`
- `frame.on(event, callback)` / `frame.off(event, callback)`
- `frame.retry(params?)`
- `frame.unmount()`

Documented events are `requestSuccess`, `requestFail`, `revealSuccess`, and
`revealFail`. Do not log failure payloads wholesale; extract only safe status or
category metadata.

## Authentication, CORS, and cookies

Choose one documented authorization approach appropriate to the customer's
backend: cookies, an authorization header, or an API key header/query parameter.
Do not hardcode credentials or put durable secrets in client JavaScript.

For credentialed requests, use a configured custom hostname, set
`withCredentials: true`, and validate server CORS and cookie attributes. The
server must explicitly allow credentials and the requesting origin. Because
`SameSite=None` weakens built-in CSRF protection, require an independent CSRF
control such as a one-time token. Never use wildcard credentialed CORS.

## Styling, formatting, and accessibility

- Apply styles through `frame.render()`; do not move revealed content into the
  parent DOM to style it.
- Treat serializer patterns as code: use fixed, reviewed patterns and test
  short, malformed, and unexpected values without recording raw results.
- Give each wrapper a stable layout to avoid frame clipping and layout shift.
- Preserve the iframe's accessible title and provide contextual labeling around
  the wrapper. Ensure copy controls have clear purpose and status feedback.

## Troubleshooting order

1. Confirm the versioned script loads and `window.VGSShow` exists; inspect CSP,
   SRI, CORS, and browser console errors without exposing secrets.
2. Confirm vault, environment, optional CNAME, and route ID match the SANDBOX
   deployment.
3. Confirm wrapper selection succeeds before `render()`.
4. Use `requestSuccess`/`requestFail` to isolate transport and authorization.
5. Use `revealSuccess`/`revealFail` to isolate response selection or rendering;
   verify `jsonPathSelector`, response content type, and route reveal phase.
6. Reproduce with one synthetic alias and one frame before restoring batching,
   serializers, images, copy controls, or credentials.

An HTTP success alone does not prove the value was safely revealed. Browser
rendering alone does not prove the route is properly authorized.

## Verification

At minimum, verify:

- script URL, SRI, CSP, and load failure behavior;
- successful synthetic SANDBOX text reveal;
- request failure and reveal failure states without sensitive output;
- selector/wrapper existence and `unmount()` cleanup;
- response selector mismatch;
- serializer and copy behavior when used;
- image/base64 behavior when used;
- CNAME, credentialed CORS, and CSRF controls when used;
- no raw revealed values enter parent-page state, logs, analytics, or test
  artifacts.
