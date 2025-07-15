# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an ESPHome implementation for connecting to Renogy Bluetooth-enabled solar charge controllers and battery management systems. The project provides two main configurations:

1. **Battery monitoring** (`renogy_batteries.yaml`) - Connects to Renogy BT-2 devices to monitor multiple batteries in a daisy-chain configuration
2. **Charge controller monitoring** (`renogy_rover.yaml`) - Connects to Renogy Rover charge controllers via BT-2 adapters

## Architecture

### Core Components

- **YAML configurations**: ESPHome device configurations that define sensors, BLE clients, and data collection intervals
- **C++ utility headers**: Low-level protocol implementations for parsing Renogy device responses
- **Bluetooth communication**: Uses ESP32 BLE client to communicate with Renogy devices over MODBUS-like protocol

### Key Files

- `renogy_batteries.yaml` - ESPHome config for battery monitoring (supports multiple batteries)
- `renogy_rover.yaml` - ESPHome config for charge controller monitoring  
- `renogy_battery_utilities.h` - C++ functions for battery data parsing and request generation
- `renogy_rover_utilities.h` - C++ functions for rover data parsing and request generation
- `secrets.yaml` - Contains WiFi credentials, MQTT settings, and other sensitive configuration

### Communication Flow

1. **BLE Connection**: ESP32 connects to Renogy device via BLE using configured MAC address
2. **Data Requests**: Interval timers send MODBUS-style requests to specific BLE characteristics
3. **Response Parsing**: C++ utility functions parse binary responses and extract sensor values  
4. **Sensor Updates**: Parsed data is published to ESPHome template sensors
5. **Home Assistant Integration**: Sensors are exposed via ESPHome API or MQTT

## Development Commands

This is an ESPHome project with no traditional build system. Development workflow:

### ESPHome Commands
```bash
# Validate configuration
esphome config renogy_batteries.yaml
esphome config renogy_rover.yaml

# Compile firmware
esphome compile renogy_batteries.yaml
esphome compile renogy_rover.yaml

# Upload to device (first time via USB)
esphome upload renogy_batteries.yaml
esphome upload renogy_rover.yaml

# Monitor logs
esphome logs renogy_batteries.yaml
esphome logs renogy_rover.yaml

# Clean build files
esphome clean renogy_batteries.yaml
esphome clean renogy_rover.yaml
```

## Configuration

### Required Configuration Updates

**Battery Configuration (`renogy_batteries.yaml`)**:
- Update `ble_mac_address` substitution with actual device MAC
- Configure `battery_id_1`, `battery_id_2`, `battery_id_3` for your battery setup
- Add/remove battery sensor definitions as needed
- Update WiFi credentials in secrets.yaml

**Rover Configuration (`renogy_rover.yaml`)**:
- Update `ble_mac_address` substitution with actual device MAC
- Configure MQTT broker settings in secrets.yaml
- Update WiFi credentials in secrets.yaml

### Protocol Details

- **Battery Protocol**: Requests battery data using MODBUS function 0x03 (read) from register 0x13B2
- **Rover Protocol**: Requests charging info from register 0x0100 (256 decimal) with 34 words
- **BLE Services**: Uses service UUID `FFD0` for writing requests, `FFF0` for reading responses
- **Data Collection**: 30-second intervals with 5-second delays between battery requests

## Code Patterns

### Adding New Battery Support
1. Add battery ID to substitutions section
2. Create template sensors for the new battery following existing patterns
3. Update interval section to include new battery request with proper delay
4. Utility functions automatically handle multiple batteries based on ID

### Extending Rover Data
1. Add new template sensors with appropriate device classes
2. Update `parse_charging_info()` function in `renogy_rover_utilities.h`
3. Use `bytes_to_int()` helper for multi-byte values
4. Use `parse_temperature()` for temperature sensors

### Debugging
- Enable DEBUG logging level in YAML configuration
- Monitor BLE connection status via binary sensors
- Check raw byte arrays in logs for protocol debugging
- Use `ESP_LOGD` statements in C++ utilities for detailed parsing info