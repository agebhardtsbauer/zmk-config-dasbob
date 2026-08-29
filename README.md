# zmk-config-dasbob

ZMK config for a DASBOB split (two nice!nanos) with a [Prospector](https://github.com/carrefinho/prospector) acting as a third, dongle-central device.

## Topology

Two supported setups, chosen by which `.uf2` you flash to the left half:

- **Standalone** (no Prospector): `dasbob_left_standalone` is BLE central and pairs directly to your computer/phone; `dasbob_right` is peripheral.
- **With Prospector**: `dasbob_left` and `dasbob_right` are both peripherals, no USB; `dasbob_dongle` + `prospector_adapter` (Prospector's xiao_ble) is central, connects to your computer over USB or BLE, and drives the display.

Only one device in the fleet can be central at a time, so switching between these means reflashing the left half.

## Building

Push this repo to GitHub and GitHub Actions (`.github/workflows/build.yml`) will build all six `.uf2` files listed in `build.yaml` as an artifact zip. Or build locally with `west` if you have a ZMK dev environment set up.

## Flashing

**Without the Prospector (two halves only, pairs directly to your computer):**

1. Flash `dasbob_left_standalone.uf2` to the left half, `dasbob_right.uf2` to the right half.
2. Pair the left half to your computer/phone like any normal BLE keyboard. The right half pairs to the left automatically over the split link.
3. If a half won't (re-)pair, flash the matching `*_settings_reset.uf2` to clear its stored bonds first, then reflash the real firmware.

**With the Prospector (dongle-central + display):**

1. Flash `dasbob_left.uf2` to the left half, `dasbob_right.uf2` to the right half. (Note: this is the *peripheral* left build, not `dasbob_left_standalone.uf2` — the two are not interchangeable, since only one device in the fleet can be BLE central at a time.)
2. Flash `dasbob_prospector_dongle.uf2` to the Prospector.
3. Put the Prospector in pairing mode and **pair the left half first, then the right half** — the peripheral battery widget lays itself out in pairing order, so pairing out of order just looks wrong on-screen (left-to-right), it won't break function.
4. If a half won't re-pair (e.g. after re-flashing), flash the matching `*_settings_reset.uf2` to clear its stored bonds first, then reflash the real firmware.

The Prospector's `dasbob_prospector_dongle.uf2` build is currently broken (tracked separately) — the standalone path above works today without it.

## Customizing

- Keymap: `config/dasbob.keymap` — starter layout adapted from the wider DASBOB ZMK community, not the official one (DASBOB ships QMK/Vial by default). Layer names carry `display-name` so they show nicely on the Prospector's layer widget.
- Prospector display options (brightness, rotation, status-screen style): `config/boards/shields/dasbob/dasbob_dongle.conf`.

## A note on the ZMK/Prospector version pin

`config/west.yml` pins `zmk` to `main` and `prospector-zmk-module` to its `feat/new-status-screens` branch — this is the only branch of the Prospector module currently compatible with current ZMK `main` (Zephyr 4.1), but the module's own README marks it **work-in-progress**. If you hit build breakage from upstream ZMK moving under you, or want a more stable base:

- Pin `zmk`'s `revision:` to a specific commit SHA instead of `main` (this repo's Actions build will then always reproduce the same firmware from the same code).
- Or switch to the Prospector module's `main` branch, which targets the older, stable ZMK `v0.3` — in `west.yml`, set `zmk`'s `revision: v0.3`, the module's `revision: main`, and change every `xiao_ble//zmk` in `build.yaml` to `seeeduino_xiao_ble`.
