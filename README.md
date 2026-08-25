# zmk-config-dasbob

ZMK config for a DASBOB split (two nice!nanos) with a [Prospector](https://github.com/carrefinho/prospector) acting as a third, dongle-central device.

## Topology

Unlike a normal 2-device split, Prospector requires **dongle-as-central**:

- `dasbob_left` (nice!nano) — peripheral, no USB
- `dasbob_right` (nice!nano) — peripheral, no USB
- `dasbob_dongle` + `prospector_adapter` (Prospector's xiao_ble) — central, connects to your computer over USB or BLE, and drives the display

The two halves no longer pair directly to your computer — the Prospector is now a required link in the chain, not an optional accessory.

## Building

Push this repo to GitHub and GitHub Actions (`.github/workflows/build.yml`) will build all five `.uf2` files listed in `build.yaml` as an artifact zip. Or build locally with `west` if you have a ZMK dev environment set up.

## Flashing

1. Flash `dasbob_left.uf2` to the left half, `dasbob_right.uf2` to the right half.
2. Flash `dasbob_prospector_dongle.uf2` to the Prospector.
3. Put the Prospector in pairing mode and **pair the left half first, then the right half** — the peripheral battery widget lays itself out in pairing order, so pairing out of order just looks wrong on-screen (left-to-right), it won't break function.
4. If a half won't re-pair (e.g. after re-flashing), flash the matching `*_settings_reset.uf2` to clear its stored bonds first, then reflash the real firmware.

## Customizing

- Keymap: `config/dasbob.keymap` — starter layout adapted from the wider DASBOB ZMK community, not the official one (DASBOB ships QMK/Vial by default). Layer names carry `display-name` so they show nicely on the Prospector's layer widget.
- Prospector display options (brightness, rotation, status-screen style): `config/boards/shields/dasbob/dasbob_dongle.conf`.

## A note on the ZMK/Prospector version pin

`config/west.yml` pins `zmk` to `main` and `prospector-zmk-module` to its `feat/new-status-screens` branch — this is the only branch of the Prospector module currently compatible with current ZMK `main` (Zephyr 4.1), but the module's own README marks it **work-in-progress**. If you hit build breakage from upstream ZMK moving under you, or want a more stable base:

- Pin `zmk`'s `revision:` to a specific commit SHA instead of `main` (this repo's Actions build will then always reproduce the same firmware from the same code).
- Or switch to the Prospector module's `main` branch, which targets the older, stable ZMK `v0.3` — in `west.yml`, set `zmk`'s `revision: v0.3`, the module's `revision: main`, and change every `xiao_ble//zmk` in `build.yaml` to `seeeduino_xiao_ble`.
