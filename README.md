# room-firmware

Firmware repository for ESP32 room/device OTA updates.

## Purpose

This repository is intended to store firmware files or firmware source used by a room/device management system. It can be paired with a dashboard such as `DeviceManager` to publish and activate OTA versions.

## Suggested Workflow

1. Build firmware for the target ESP32 board.
2. Create a versioned firmware file.
3. Upload or attach the firmware to a release.
4. Activate the version from the management dashboard.
5. Devices download and install the active firmware through OTA.

## Recommended Repository Structure

```text
firmware/       Source code or compiled firmware files
releases/       Versioned firmware artifacts
docs/           Board notes and OTA instructions
```

## Documentation To Add

- ESP32 board model.
- Build tool: Arduino IDE, PlatformIO, or ESP-IDF.
- Required libraries.
- Wi-Fi and OTA configuration.
- Firmware versioning rules.
- Rollback plan for failed OTA updates.

## Related Project

- [DeviceManager](https://github.com/giangbc2k4/DeviceManager)
