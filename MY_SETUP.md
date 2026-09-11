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
2. Flash **left half**: bootloader mass-storage mode ([MoErgo's guide](https://docs.moergo.com/glove80-user-guide/customizing-key-layout/#loading-new-zmk-firmware-onto-your-glove80)) → copy the `.uf2` onto it.
3. Flash **right half**: same as above.

### Factory reset + re-pair (required since firmware version changed from stock)

Full procedure: [MoErgo troubleshooting guide](https://docs.moergo.com/glove80-user-guide/troubleshooting/#configuration-factory-reset-and-re-pairing-left-and-right-halves).

Order matters — reset each half individually (powered off otherwise), *then*
re-pair by powering both on together:

4. Reset **left half**: power off both halves → hold `Magic + 3` → power on
   **left only** → hold 5s → power off left.
5. Reset **right half**: power off both halves → hold `PgDn + 8` → power on
   **right only** → hold 5s → power off right.
6. Re-pair: power on **both halves at the same time**.
7. Toggle RGB on (`Magic + T`), confirm both halves light up, toggle off.
8. Leave both halves powered on for **at least 1 minute** before switching
   off, so the new pairing bond persists to non-volatile memory.

Why: flashing firmware doesn't touch each half's stored Bluetooth pairing
keys. A firmware/version change can shift the BLE bond structure, so both
halves need to be wiped back to a clean state and re-paired together —
wiping only one half causes a mismatch since the bond is mutual.

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
