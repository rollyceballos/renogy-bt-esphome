# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an ESPHome implementation for connecting to a Renogy Rover solar charge controller. The project is specifically configured for a **Heltec WiFi Kit 32** board and uses its onboard OLED display to show real-time status information.

The primary configuration is in `renogy_rover.yaml`.

## Architecture

### Core Components

- **YAML Configuration**: The main ESPHome device configuration is `renogy_rover.yaml`. It defines sensors, the BLE client, the OLED display, and data collection logic.
- **C++ Utility Header**: `renogy_rover_utilities.h` provides the low-level protocol implementation for parsing Renogy device responses and generating requests.
- **Bluetooth Communication**: Uses the ESP32 BLE client to communicate with the Renogy device over a MODBUS-like protocol.
- **OLED Display**: Utilizes the Heltec board's built-in SSD1306 OLED screen to display status.

### Key Files

- `renogy_rover.yaml` - ESPHome config for Rover charge controller monitoring.
- `renogy_rover_utilities.h` - C++ functions for Rover data parsing and request generation.
- `secrets.yaml` - (User-created) Contains WiFi credentials, MQTT settings, and the Renogy device's BLE MAC address.

### Communication & Display Flow

1.  **BLE Connection**: The ESP32 connects to the Renogy Rover via BLE using the configured MAC address.
2.  **Data Requests**: An interval timer sends MODBUS-style requests to the Rover every 5 seconds.
3.  **Response Parsing**: C++ utility functions parse the binary responses and extract sensor values.
4.  **Sensor Updates**: Parsed data is published to ESPHome template sensors.
5.  **Status Updates**: WiFi, BLE, and MQTT connection statuses are updated in the background.
6.  **Screen Timeout**: The OLED display automatically turns off after 30 seconds of inactivity to prevent burn-in.
7.  **Wake on Button Press**: Pressing the onboard "PRG" button (GPIO0) wakes the display and shows the latest data.

## Development Commands

This is an ESPHome project. The development workflow uses the ESPHome command-line tool:

```bash
# Validate configuration
esphome config renogy_rover.yaml

# Compile firmware
esphome compile renogy_rover.yaml

# Upload to device (first time via USB)
esphome run renogy_rover.yaml

# Monitor logs
esphome logs renogy_rover.yaml

# Clean build files
esphome clean renogy_rover.yaml
```

## Configuration

### Required Configuration Updates

**Rover Configuration (`renogy_rover.yaml` via `secrets.yaml`)**:
- Create a `secrets.yaml` file in the root directory.
- Update `wifi_ssid` and `wifi_password` with your network credentials.
- Update `renogy_rover_ble_mac` with your device's actual MAC address.
- Configure MQTT broker settings (`mqtt_host`, `mqtt_username`, `mqtt_password`).

### Protocol Details

- **Rover Protocol**: Requests charging info from register 0x0100 (256 decimal) with 34 words.
- **BLE Services**: Uses service UUID `FFD0` for writing requests and `FFF0` for reading responses.
- **Data Collection**: A 5-second interval is used to request data from the Rover.

## Code Patterns

### Extending Rover Data
1.  Add new template sensors in `renogy_rover.yaml` with appropriate device classes.
2.  Update the `HandleRoverData()` function in `renogy_rover_utilities.h` to parse the new data.
3.  Use the `bytes_to_int()` helper for multi-byte values.
4.  Update the display lambda in `renogy_rover.yaml` to show the new sensor data.

### Debugging
- Enable `DEBUG` logging level in the `logger` section of `renogy_rover.yaml`.
- Monitor the BLE connection status via the `Renogy BLE Presence Rover` binary sensor.
- Check raw byte arrays in the logs for protocol debugging by uncommenting the log lines in the `lambda` for the `renogy_rover_esp32_sensor`.
- Use `ESP_LOGD` statements in C++ utilities for detailed parsing info.