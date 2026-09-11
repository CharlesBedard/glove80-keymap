# My Glove80 setup

Personal notes for configuring this keymap via MoErgo's web layout editor.
See `README.md` for the full upstream guide — this is just my checklist.

## Layout editor

https://my.moergo.com/glove80/#/layout/user/909a8599-903f-44f3-a51e-5130ecdf2999

Log in, clone it to my account, then edit.

## Custom Defined Behaviors

Paste at the top of the "Custom Defined Behaviors" text box:

```h
#define OPERATING_SYSTEM 'M' // macOS

#define SPACE_FORGIVENESS // for lingering taps on thumb letter R in Enthium
#define THUMB_HOLDING_TIME 200 // prefer typing over faster layer activation
#define RIGHT_INDEX_STREAK_DECAY 80 // faster HRM shift cooldown (gcA alias)

#define ENABLE_MOUSE_KEYS // uncomment in the generated snippet if not already
```

Base layer: QWERTY (already set as layer #0 in this repo's history; confirm via
drag & drop in the editor if a different base layout is wanted).

## Firmware version / Advanced Configuration

Before building (Settings tab):

- Set firmware version to **PR36** (needed for per-key RGB and community mouse
  keys). Alternatively `community.pr36.mouse-keys` from the dropdown.

In **Advanced Configuration** tab, tick "Overridden" + "y" for:

- `EXPERIMENTAL_RGB_LAYER` — per-key RGB layer indication
- `HID_POINTING` — mouse emulation (only if not already enabled by PR36 default)

## Build & flash

1. Build firmware, download the `.uf2`.
2. First flash: bootloader mass-storage mode on **both halves** — see
   [MoErgo's guide](https://docs.moergo.com/glove80-user-guide/customizing-key-layout/#loading-new-zmk-firmware-onto-your-glove80).
3. Since this is a firmware version change from stock, after flashing both
   halves: do a
   [factory reset + re-pair](https://docs.moergo.com/glove80-user-guide/troubleshooting/#configuration-factory-reset-and-re-pairing-left-and-right-halves)
   on both halves, then cycle RGB effects on/off once for the new firmware to
   take effect.

## After it's working: export for CI

Once flashed and confirmed working on hardware, export the local config so it
can be ported into a self-hosted build later:

1. Layout Editor → Settings → Experimental Settings → enable "Enable local
   config".
2. Back on the Edit tab, click "Download" to get the JSON export.

This JSON is the known-good source to port into a fork of
[`moergo-sc/glove80-zmk-config-west`](https://github.com/moergo-sc/glove80-zmk-config-west)
(pinned to the PR36 branch, `rgb-layer-24.12`, in `config/west.yml`) for
automated GitHub Actions builds later, instead of reconstructing the keymap
from scratch.
