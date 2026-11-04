# Bluetti2MQTT Plus – AC500 Secure BLE (v4.0.0)

Standalone Home Assistant MQTT add-on for **Bluetti AC500 with encrypted Bluetooth**.

This v4 add-on is installed alongside, not instead of, the legacy `bluetti2mqtt` add-on (v3.0.3), which remains available for previously supported models and unencrypted Bluetooth.

## Features
- Independent bridge built on `bluetti-bt-lib==0.1.8`; no semitop7 dependency at build time or runtime.
- Encrypted Bluetooth read and write support for compatible AC500.
- MQTT Discovery and continuity of migrated device topic names.
- `grid_charge_amps`, SOC charging range, and other verified register commands when `allow_writes` is enabled.
- Detection and monitoring of supported B300 battery packs including 16-cell voltage arrays.
- General telemetry polling defaults to 15 seconds (best effort), battery details to 600 seconds, full pack scan to 3600 seconds.
- Retained MQTT commands are ignored on subscription to prevent unintended replay.

## Installation
Add the repository to the Home Assistant Add-on Store:
`https://github.com/LinusIta/bluetti2mqtt_plus`

Select **Bluetti2MQTT Plus - AC500 Secure BLE** for AC500 devices with encrypted BLE. For unencrypted models, continue using **Bluetti2MQTT Plus** (legacy 3.0.3).

Configure the AC500 Bluetooth MAC address, device serial, MQTT broker and credentials where needed; enable `allow_writes` explicitly for command support. Running both bridges against the same device/MQTT identity can create duplicate entities or conflicting commands.

## Upgrade notice
The v4 add-on retains the tested `bluetti2mqtt_plus_v4_preview` internal slug for compatibility with local dev12 installations. Existing local add-on installations do not necessarily upgrade in-place from a separate GitHub repository installation; **back up configuration and confirm the installed add-on identity before migrating.**

## Limitations
- Focused on AC500; other Bluetti models are not claimed supported by this implementation.
- Bluetooth is best effort, may experience intermittent transport timeouts and reconnects.
- Modbus TCP is not included in v4.0.0; hybrid transport is a separate future development.
