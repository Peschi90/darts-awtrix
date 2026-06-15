# DARTS-AWTRIX
[![Downloads](https://img.shields.io/github/downloads/lbormann/darts-awtrix/total.svg)](https://github.com/lbormann/darts-awtrix/releases/latest)

Darts-awtrix controls your [AWTRIX 3](https://blueforcer.github.io/awtrix3/) installation(s) according to the state of an [autodarts.io](https://autodarts.io) game.
A running instance of [darts-caller](https://github.com/lbormann/darts-caller) is needed to relay throws/events from autodarts.io to this application.

Communication with AWTRIX is done via the [AWTRIX HTTP API](https://blueforcer.github.io/awtrix3/#/api) (`/api/notify`, `/api/settings`, `/api/power`) — no MQTT.

This project is a sibling of [darts-pixelit](https://github.com/lbormann/darts-pixelit) and uses the same argument layout plus the same signed-extension authentication against darts-caller.


## COMPATIBILITY

| Variant | Support |
| ------------- | ------------- |
| X01 | :heavy_check_mark: |
| Cricket | :heavy_check_mark: |
| Random Checkout | :heavy_check_mark: |
| Bermuda / Shanghai / Gotcha / ATC / Count Up / Segment Training | |


## Hardware

Any AWTRIX 3 device works (e.g. Ulanzi TC001 or a self-built ESP32 + 8x32 WS2812 matrix flashed with [AWTRIX 3 firmware](https://blueforcer.github.io/awtrix3/)). The device must be reachable over HTTP from the machine running darts-awtrix.


## INSTALL INSTRUCTION

### Desktop-OS:

- If you're running a desktop-driven OS it's recommended to use [darts-hub](https://github.com/lbormann/darts-hub) as it takes care of starting, updating, configuring and managing multiple apps.

### Headless-OS:

- Download the appropriate executable from the release section.


## RUN IT

### Prerequisite

- A running [darts-caller](https://github.com/lbormann/darts-caller) (latest version).
- A reachable AWTRIX 3 device (firmware up-to-date).

### Run by executable (Windows example)

    C:\Downloads\darts-awtrix.exe -AEPS "192.168.1.50" -TP "C:\AWTRIX\templates"


### Arguments

| Group | Arguments |
| -- | -- |
| Connection | `-CON`, `-AEPS`, `-TP` |
| Display settings (one-shot at startup) | `-BRI`, `-ABRI`, `-TEFF`, `-TSPEED`, `-TCOL`, `-SSPEED`, `-UPC`, `-DUR`, `-PWR`, `-ATR` |
| Game/lobby events | `-HFO`, `-HF`, `-AS`, `-IDE`, `-GS`, `-MS`, `-G`, `-M`, `-B`, `-PJ`, `-PL` |
| Scores | `-S{0-180}`, `-A{1-12}` |
| Misc | `-DEB` |

All event arguments accept *multiple* values; each value is one effect-definition (template name + optional parameters).

#### Connection

| Argument | Description |
| -- | -- |
| `-CON` / `--connection` | Address of darts-caller. Default `127.0.0.1:8079`. |
| `-AEPS` / `--awtrix_endpoints` | **Required.** One or more AWTRIX IPs/hostnames, e.g. `192.168.1.50 192.168.1.51`. |
| `-TP` / `--templates_path` | **Required.** Absolute path to folder containing your `*.json` templates. Must NOT be a sub-folder of the application. |

#### Display settings

These are pushed to each AWTRIX endpoint **once** at startup via `/api/settings` and `/api/power`. Only the fields you actually pass are sent.

| Argument | AWTRIX field | Description |
| -- | -- | -- |
| `-BRI` / `--effect_brightness` | `BRI` 0..255 | Global matrix brightness. Default 100. |
| `-ABRI` / `--auto_brightness` | `ABRI` bool | `1` = automatic brightness, `0` = stick to `BRI`. If unset and `-BRI` is provided, `ABRI` is forced to `false` so `BRI` actually sticks. |
| `-TEFF` / `--transition_effect` | `TEFF` 0..10 | Transition between apps: 0 Random, 1 Slide, 2 Dim, 3 Zoom, 4 Rotate, 5 Pixelate, 6 Curtain, 7 Ripple, 8 Blink, 9 Reload, 10 Fade. |
| `-TSPEED` / `--transition_speed` | `TSPEED` int ms | Transition speed. |
| `-TCOL` / `--text_color` | `TCOL` color | Global text color (hex `#FFAA00` or `r,g,b`). |
| `-SSPEED` / `--scroll_speed` | `SSPEED` int % | Text scroll speed as percentage of native. |
| `-UPC` / `--uppercase` | `UPPERCASE` bool | Force uppercase display text. |
| `-DUR` / `--notify_duration` | n/a | Default notification duration in seconds. `0` (default) = **endless**: the event is dispatched as a persistent Custom App (`darts_event`) and stays on the matrix until the next event overrides it. Any positive value pushes the event as a regular `/api/notify` with that duration. Priority: runtime override `d:X` > template's `duration` field > `-DUR`. |
| `-PWR` / `--matrix_power` | n/a | If `1`, sends `POST /api/power {power:true}` at startup. Default `1`. |
| `-ATR` / `--auto_transition` | `ATRANS` | Toggle automatic app rotation. `0` keeps the current app on screen indefinitely — the cleanest way to keep the `darts_idle` Custom App visible between throws without rebooting the device. `1` re-enables rotation. Takes effect immediately. |

#### High-finish

| Argument | Description |
| -- | -- |
| `-HFO` / `--high_finish_on` | Threshold (2..170): a `game-won` / `match-won` whose score is `>= -HFO` plays the `-HF` chain instead of the regular `-G` / `-M`. |
| `-HF` | Effect chain played on a high-finish. |

#### Per-event effects

| Argument | Triggered when |
| -- | -- |
| `-AS` | Application start |
| `-IDE` | Darts pulled out of the board |
| `-GS` | Game started |
| `-MS` | Match started |
| `-G` | Game won |
| `-M` | Match won |
| `-B` | Bust |
| `-PJ` | Player joined lobby |
| `-PL` | Player left lobby |
| `-S{0..180}` | Specific score thrown |
| `-A{1..12}` | Score area (`first-last template1 template2 ...`) |

### Effect definition syntax

```
template[|t:Text{}with{}spaces][|d:Duration_s][|b:Brightness][|p:ScrollSpeed][|u:TextCase][|e:Endpoint_idxs][|i:Icon][|c:Color][|s:Sound]
```

| Param | Description |
| -- | -- |
| `t:` | Overrides the `text` field of the template. `{}` expands to a space. Variables: `{playername}`, `{score}`, `{points-left}`, `{p1-points-left}` ... `{p6-points-left}`, `{game-mode}`, `{game-mode-extra}`. |
| `d:` | Per-effect duration in **seconds**. Highest priority — overrides both the template's `duration` and the global `-DUR`. `0` = endless (Custom App slot). |
| `b:` | Brightness (`0..255`). Highest priority — overrides the template's `brightness` field and the global `-BRI`. AWTRIX has no per-notification brightness, so this is written via `/api/settings` (de-duplicated per endpoint) and persists on the device until changed again. |
| `p:` | Scroll speed in percent of the original (`0..N`). Highest priority — overrides the template's `scrollSpeed` field and the global `-SSPEED`. Per-notification only. |
| `u:` | Uppercase / text case: `0` = use global, `1` = force uppercase, `2` = render as sent. Highest priority — overrides the template's `textCase` field and the global `-UPC`. Per-notification only. |
| `e:` | Comma-separated 0-based indices into `-AEPS` (target a subset of devices). |
| `i:` | Overrides the `icon` field of the template. Use a numeric icon ID from the official [LaMetric / AWTRIX icon gallery](https://developer.lametric.com/icons) (the same gallery AWTRIX 3 uses — see the [AWTRIX icon docs](https://blueforcer.github.io/awtrix3/#/icons)), or the filename of a custom icon you uploaded to the device. |
| `c:` | Overrides the `color` field (`#FF0000` / `FF0000` / `255,0,0`). Highest priority — overrides the template's `color` field and the global `-TCOL`. |
| `s:` | Overrides the `sound` field (RTTTL file name without extension, or 4-digit DFplayer track id). |

### Examples

| Argument | Example |
| -- | -- |
| `-AS` | `appstart\|t:Lets{}Play{}Darts` |
| `-G` | `gamewon\|t:Gameshot!{}{playername}{}-{}{score}` |
| `-A1` | `0-15` `score\|t:{score}\|c:#FF0000` |
| `-B` | `busted\|t:BUSTED{}{playername}` |
| `-S180` | `score180\|t:180\|s:beep\|e:0` |

### Text variables

| Argument | Variables |
| -- | -- |
| `-HF` | `{playername}`, `{score}`, `{p1-points-left}` .. `{p6-points-left}` |
| `-AS` | (none) |
| `-IDE` | `{playername}`, `{points-left}`, `{p1-points-left}` .. `{p6-points-left}` |
| `-MS`, `-GS` | `{game-mode}`, `{game-mode-extra}`, `{playername}`, `{p1-points-left}` .. `{p6-points-left}` |
| `-G`, `-M` | `{playername}`, `{score}`, `{p1-points-left}` .. `{p6-points-left}` |
| `-B` | `{playername}`, `{p1-points-left}` .. `{p6-points-left}` |
| `-PJ`, `-PL` | `{playername}` |
| `-S{0..180}`, `-A{1..12}` | `{playername}`, `{score}`, `{p1-points-left}` .. `{p6-points-left}` |

### Endpoint-Targeting

With multiple AWTRIX devices in `-AEPS`, `e:` selects which receive a given notification. Indices are 0-based.

```
-AEPS "192.168.1.50" "192.168.1.51" "192.168.1.52"
-S180 "fire|e:0" "score180|t:180|e:1,2"
```

| Effect | Target |
| -- | -- |
| `fire` | all devices |
| `fire\|e:0` | only first device |
| `fire\|e:0,2` | first and third device |


## Templates

A template is any JSON object accepted by AWTRIX `POST /api/notify`. See the [AWTRIX JSON properties](https://blueforcer.github.io/awtrix3/#/api?id=json-properties) for the full list. Useful keys:

- `text` (string or colored-fragment array)
- `color`, `background`, `rainbow`, `gradient`, `blinkText`, `fadeText`
- `icon` (ID or filename of an icon uploaded to the device)
- `pushIcon` (0/1/2)
- `duration` (seconds), `repeat`
- `wakeup` (wake matrix from sleep)
- `hold`, `stack`
- `sound`, `rtttl`, `loopSound`
- `bar`, `line`, `progress`, `progressC`, `progressBC`
- `draw` (array of drawing instructions: `dp`, `dl`, `dr`, `df`, `dc`, `dfc`, `dt`, `db`)
- `effect`, `effectSettings`, `overlay`

Sample templates ship under `community/templates/`. Sounds (`s:`) and icons (`i:`) refer to assets that must already live on the device.


## !!! IMPORTANT !!!

This application requires a running instance of [darts-caller](https://github.com/lbormann/darts-caller).


## Credits & Thanks

A huge thank-you to:

- **[Stephan Müller (Blueforcer)](https://github.com/Blueforcer)** — creator and maintainer of [AWTRIX 3](https://github.com/Blueforcer/awtrix3), the open-source firmware that powers every supported device. Without his work this project would not exist.
- The entire **[AWTRIX 3 community](https://blueforcer.github.io/awtrix3/)** for the excellent [HTTP API](https://blueforcer.github.io/awtrix3/#/api), the [icon ecosystem](https://blueforcer.github.io/awtrix3/#/icons) and the active support around the firmware.
- **[LaMetric](https://developer.lametric.com/icons)** for the public icon gallery that AWTRIX 3 builds upon.
- **[lbormann](https://github.com/lbormann)** for [darts-caller](https://github.com/lbormann/darts-caller) and the sibling project [darts-pixelit](https://github.com/lbormann/darts-pixelit) that this application is modeled after.
