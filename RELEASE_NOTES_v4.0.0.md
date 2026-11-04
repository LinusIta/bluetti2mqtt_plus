# Bluetti2MQTT Plus v4.0.0 — AC500 Secure BLE

## Stable release

A new **standalone Home Assistant add-on** for **Bluetti AC500 devices requiring encrypted Bluetooth** after firmware updates.

### Features
- Encrypted Bluetooth transport with pinned `bluetti-bt-lib==0.1.8`.
- Independent MQTT bridge: **no semitop7 dependency** at build time or runtime.
- Home Assistant MQTT Discovery, preserving the AC500 topics and aliases used during v4 dev12 testing.
- Regular AC500 telemetry polling (approximately 15 seconds under normal conditions; best effort).
- Detected B300 packs, per-pack SOC, voltage and 16-cell voltage arrays.
- Verified encrypted BLE writes for charging current and charging SOC range, gated by `allow_writes`.
- Ignores retained MQTT command messages received at subscription to prevent replayed writes.
- Includes the same Home Assistant icon and logo artwork as the legacy 3.0.3 add-on.

### Compatibility — IMPORTANT

Both add-ons remain available in this repository:

- **Legacy 3.0.3** — `bluetti2mqtt/` — existing users with unencrypted BLE or other previously supported Bluetti models. The legacy implementation is unchanged.
- **AC500 Secure BLE 4.0.0** — `bluetti2mqtt_ac500_ble/` — AC500 devices with encrypted Bluetooth.

**Do not migrate legacy users automatically.** AC500 Secure BLE is not claimed to support other Bluetti models. Do not run both bridges publishing to the same MQTT identity.

### Installation

Add `https://github.com/LinusIta/bluetti2mqtt_plus` to the Home Assistant add-on repositories and select **Bluetti2MQTT Plus - AC500 Secure BLE**. Back up your existing settings and enter your BLE MAC, device serial, broker settings, and optional write permission. Existing local dev12 installations may require a manual migration due to add-on repository identity.

### Known limitations

- One AC500 + B300 configuration was field-tested; there is no guarantee for all firmware variants.
- BLE sessions may intermittently time out or fail signature checks; subsequent polling cycles retry.
- This stable release does not yet implement Modbus TCP / hybrid transport.
- Prepared from the working dev12 version, without a new independent device-level test following the release metadata changes.
