# NeewerLite

A description of this package.

## Local HTTP API

NeewerLite starts an HTTP server (Swifter) on `127.0.0.1:18486`.

### User-Agent requirement

All requests must include a `User-Agent` header that starts with:

`neewerlite.sdPlugin/`

Otherwise the server returns `401 Unauthorized`.

### Light selection

Most endpoints accept either:

1) JSON body field `lights: [String]` (existing behavior)

or (for the *Delta* endpoints below)

1) Query parameter `?light=...`

The `light` selector supports:

- exact matches (case-insensitive) against: `userLightName`, `rawName`, or `identifier`
- comma-separated lists: `Front,Back,Side`
- wildcard prefix with `*`: `NEEWER-*`
- combined: `Front*,Back,NEEWER-*`

Wildcard matching is prefix-based.

### Relative adjustments (INC/DEC style) via Delta endpoints

These endpoints adjust the *current* device value by adding `delta` (positive or negative), without the client having to send an absolute value.

Each endpoint supports either:

- Query-style:

  `POST /<endpoint>?light=NEEWER-*&delta=5`

- JSON-style:

  `POST /<endpoint>` with body:

  ```json
  { "lights": ["NEEWER-*", "KeyLight"], "delta": -10 }
  ```

#### `POST /brightnessDelta`

Adjusts brightness by `delta` and clamps to `0…100`.

Example:

`POST http://127.0.0.1:18486/brightnessDelta?light=NEEWER-*&delta=5`

#### `POST /temperatureDelta`

Adjusts CCT by `delta` and clamps to the device range returned by `CCTRange().minCCT…maxCCT`.

Example:

`POST http://127.0.0.1:18486/temperatureDelta?light=NEEWER-*&delta=100`

#### `POST /hueDelta`

Adjusts hue by `delta` (degrees). The resulting hue wraps around to `0…<360`.

Only applies to RGB-capable lights.

Example:

`POST http://127.0.0.1:18486/hueDelta?light=NEEWER-*&delta=10`

#### `POST /satDelta`

Adjusts saturation by `delta` and clamps to `0…100`.

Only applies to RGB-capable lights.

Example:

`POST http://127.0.0.1:18486/satDelta?light=NEEWER-*&delta=-5`

### Implementation reference

See route implementation in [`NeewerLite/Server.swift`](NeewerLite/Server.swift:41).

### URL-scheme-like HTTP endpoints (power)

For HTTP clients that want the same naming as the `neewerlite://...` commands, these endpoints are available:

- `POST /turnOnLight`
- `POST /turnOffLight`
- `POST /toggleLight`

They accept the same `?light=...` selector (including wildcards):

`POST http://127.0.0.1:18486/turnOnLight?light=NEEWER-*`

## URL Scheme commands (neewerlite://)

NeewerLite also supports a custom URL scheme:

`open "neewerlite://<command>?..."`

This is handled in [`AppDelegate.handleURLEvent()`](NeewerLite/AppDelegate.swift:1142) via the internal [`CommandHandler.execute()`](NeewerLite/Model/Command.swift:212).

### Basic commands

- `open "neewerlite://turnOnLight"` (all lights)
- `open "neewerlite://turnOnLight?light=left"` (one or more lights)
- `open "neewerlite://turnOffLight"`
- `open "neewerlite://toggleLight"`
- `open "neewerlite://scanLight"`
- `open "neewerlite://listLights"` (prints into app log)

Light selection supports comma-separated lists and `*` wildcards (prefix match), same as described above.

### Absolute value commands

- Brightness (0…100)
  - `open "neewerlite://setBrightness?light=left&Brightness=75"`

- Temperature / CCT
  - `open "neewerlite://setTemperature?light=left&CCT=4500"`

- HSI
  - `open "neewerlite://setHue?light=left&HUE=180"`
  - `open "neewerlite://setSaturation?light=left&Saturation=80"`

### Relative (Delta) commands

The following commands use `delta` (positive/negative). Example:

`open "neewerlite://brightnessDelta?light=left&delta=15"`

- `brightnessDelta` clamps to `0…100`
- `temperatureDelta` clamps to device CCT range
- `hueDelta` wraps around `0…<360` (RGB lights only)
- `satDelta` clamps to `0…100` (RGB lights only)
