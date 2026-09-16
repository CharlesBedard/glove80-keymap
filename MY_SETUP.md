# My Glove80 setup

Personal notes for configuring this keymap via MoErgo's web layout editor.
See `README.md` for the full upstream guide — this is just my checklist.

## Layout editor

https://my.moergo.com/glove80/#/layout/user/909a8599-903f-44f3-a51e-5130ecdf2999

Log in, clone it to my account, then edit.

## Custom Defined Behaviors

As of the full redesign (see below), this is no longer hand-retyped — the
whole layout including these defines is uploaded as one JSON file. For
reference, the defines baked into that file:

```h
#define OPERATING_SYSTEM 'L' // Linux (base); macOS overlay applied at runtime

#define SPACE_FORGIVENESS // for lingering taps on thumb letter R in Enthium
#define THUMB_HOLDING_TIME 200 // prefer typing over faster layer activation
#define RIGHT_INDEX_STREAK_DECAY 80 // faster HRM shift cooldown (gcA alias)

#define ENABLE_MOUSE_KEYS // uncomment in the generated snippet if not already
```

Base layer: QWERTY (layer #0).

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

## Visual indicators (RGB)

Three separate signals I wanted; here's the actual status of each:

- **Which layer I'm on** — solved, works out of the box. With
  `EXPERIMENTAL_RGB_LAYER` enabled, hold Magic and tap **G** to cycle RGB
  effects (`RGB_EFF`) forward through: Solid → Breathe → Spectrum → Swirl →
  **Layer Indicators** (this last one only exists because the flag is on).
  Confirmed on hardware: holding different layer keys (Number/Symbol/Cursor/
  etc.) visibly changes the color. No extra config needed — MoErgo's default
  build already includes color mappings, contrary to what the PR36 firmware
  source alone suggested (it only shows the *rendering engine*, not where a
  default color map is set — evidently the shipped default config provides
  one anyway).

- **Which computer/BT profile is active** — no persistent indicator exists
  for this in MoErgo's firmware. Went with flash-on-demand instead: Magic + T
  briefly flashes the current profile/battery status. A persistent option
  would require duplicating the base layer per computer and tying each BT
  profile switch to also jump to its own colored layer — decided against it
  since it reintroduces the layer bloat already cleaned up.

- **Which OS mode is active** — originally a manual toggle (Magic + Backslash
  / Grave), confirmed NOT covered by default RGB layer colors (toggling it
  produced no visible change, since it's a "hidden" layer with no key
  remapping of its own). As of the full redesign below, this manual toggle no
  longer exists at all — OS mode now switches automatically as part of the
  BT profile switch, so there's nothing left to indicate separately. Adding a
  persistent color for it would still require hand-authoring a
  `zmk,underglow-layer` devicetree node (confirmed real from MoErgo's
  `moergo-sc/zmk` PR #36 source, untested territory, no existing example to
  follow) — low priority now that the toggle itself is gone.

## Full redesign (2026-09-16)

Rebuilt from scratch to optimize for Omarchy/Hyprland + Neovim use and
seamless Mac/Linux switching. The finished file lives at
`~/Downloads/glove80-redesigned-2026-09-16.json` and is also committed as
this repo's `keymap.json`. To apply it: Layout Editor → Settings →
Experimental Settings → enable "Enable local config" → back on Edit tab →
"Upload" this JSON file (confirmed real via README's own "Mirroring
horizontally" section, which describes uploading an edited JSON back in).

**What changed:**

- Deleted 15 unreachable layers (Dvorak, Colemak, Enthium, the 8 per-finger
  pinky-shift layers, Emoji, World, Factory) and renumbered everything.
  Kept Gaming even though unreachable, per request. All raw-integer layer
  references (`&to N`, `&tog N`) were hand-verified against the new indices;
  symbolic `LAYER_X` references auto-resolve since MoErgo's compiler
  generates those `#define`s from `layer_names` order.
- **Seamless Mac/Linux switching**: BT profile 0 = Linux, profile 1 = Mac.
  Switching profile (on the Magic layer, same keys as before) now also
  resets shortcut mode in the same keypress, via two new macros
  (`bt_seamless_linux`, `bt_seamless_mac`) using `&to 0` + `&tog <macOS
  layer>`. Verified against ZMK's actual `zmk_keymap_layer_to()` source
  that `&to` deterministically clears every layer before activating its
  target, so this can't desync the way a bare toggle could. The old
  standalone manual macOS-mode toggle key is gone — no longer needed.
  `OPERATING_SYSTEM` is set back to `'L'` (Linux) as the compile-time base
  (governs HRM GACS ordering + Unicode input method), since Linux is now
  the primary/default profile.
  - **Known limitation**: this only covers the pre-baked shortcuts already
    built into the keymap (copy/paste/undo/redo/find/select, in the Cursor
    layer's `_COPY`/`_PASTE`/etc. macros). A raw, ad-hoc `Ctrl+something`
    typed via bare home-row-mods (not one of those pre-baked macros) will
    still send literal Ctrl even while on the Mac profile, since HRM's
    per-finger modifier assignment is a compile-time constant, not part of
    the runtime macOS-mode overlay. Fixing that fully would mean extending
    the overlay to the raw HRM behaviors themselves — bigger, unverified
    devicetree work, not attempted here.
- **Thumb cluster redesign** (Space/Backspace on left/right T4 kept exactly
  where they were, just fixed a swapped label — the description text had
  Cursor/Symbol backwards from the actual bound layer):
  - Right T2 (was a redundant duplicate of left T2's Lower access): now
    Super+Tab (next Hyprland workspace).
  - Right T3 (was mouse-scroll-up, deprioritized per request): now
    Alt+Shift+Tab (previous window), pairing with T6 the same way left T3/T6
    already pair for Page Up/Down.
  - Right T6 (was mouse-scroll-down): now Alt+Tab (next window).
  - Mouse layer itself untouched — still reachable exactly as before.
- Screenshot (`PSCRN`) added to the Function layer (grouped with the
  existing one-thumb-hold volume/brightness/media cluster), in addition to
  its existing spot on the System layer.
- QWERTY's old Emoji/World hold-keys (outer columns, both hands) simplified
  to plain sticky one-shot Shift, since those layers are gone — kept the
  useful half of what those keys did.
- **Not done**: baking Super into the Number layer's digit row to shrink the
  Super+workspace-number chord from 3-way (pinky Win + Number-layer thumb +
  digit) to 2-way. Tried this, caught it before shipping — it would break
  plain number typing on that exact layer (every digit press while holding
  Number would send Super+digit instead of the digit itself). Left as a
  3-way hold; fixing it properly would need a dedicated key/layer not
  already spoken for by the thumb redesign above.

First build attempt with this file may need a fix-up round — this is a
first-of-its-kind edit to this specific JSON format and MoErgo's web app
(which assembles this JSON into the final buildable keymap) is closed
source, so this couldn't be test-compiled ahead of time.

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
