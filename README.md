# renogy-bt-esphome
ESPHome implementation to pull data from a Renogy Rover solar charge controller using a Heltec WiFi Kit 32 board.

## Setup
- `renogy_rover.yaml` is configured to connect to a Renogy BT device (tested on a BT-2) and display real-time information on the Heltec WiFi Kit 32's onboard OLED screen.

- `renogy_rover_utilities.h` contains methods to do the payload creation and response parsing for communicating with the BT device. This file shouldn't require any editing in order to get things running.

### Configuration Setup

Before compiling, you'll need to create a `secrets.yaml` file in the project root with your specific configuration:

```yaml
# WiFi Configuration
wifi_ssid: "Your_WiFi_Name"
wifi_password: "Your_WiFi_Password"

# MQTT Configuration
mqtt_host: "192.168.1.100"  # Your MQTT broker IP
mqtt_username: "your_mqtt_user"
mqtt_password: "your_mqtt_password"

# Renogy device MAC address
renogy_rover_ble_mac: "AA:BB:CC:DD:EE:FF"  # Your Rover BT device MAC
```

**Finding your device MAC address:**
- Use a Bluetooth scanner app on your phone
- Look for devices named "BT-TH-" followed by numbers
- Copy the MAC address (format: AA:BB:CC:DD:EE:FF)

### Getting started with ESPHome
If you don't yet have any experience with ESPHome, I recommend looking here for guidance: https://esphome.io/guides/getting_started_command_line

Once ESPHome is installed, you can run commands such as these from the root of this project to compile and upload the firmware to your Heltec WiFi Kit 32 board:

- `esphome compile renogy_rover.yaml` - Compiles the firmware so you can test your changes without needing a board connected.
- `esphome run renogy_rover.yaml` - Compiles and uploads the firmware to a connected board.
- `esphome logs renogy_rover.yaml` - View the logs from your device.

### Disclaimer

This is not an official library endorsed by the device manufacturer. Renogy and all other trademarks in this repo are the property of their respective owners and their use herein does not imply any sponsorship or endorsement.

## References
 - [Olen/solar-monitor](https://github.com/Olen/solar-monitor)
 - [corbinbs/solarshed](https://github.com/corbinbs/solarshed)
 - [Rover 20A/40A Charge Controller—MODBUS Protocol](https://github.com/cyrils/renogy-bt/files/12787920/ROVER.MODBUS.pdf)
 - [Lithium Iron Battery BMS Modbus Protocol V1.7](https://github.com/cyrils/renogy-bt/files/12444500/Lithium.Iron.Battery.BMS.Modbus.Protocol.V1.7.zh-CN.en.1.pdf)
 - [The original Renogy BT project](https://github.com/cyrils/renogy-bt) - This current project is effectively a port of cyrils' work to make it run on ESPHome. None of this would be possible without the groundwork that was laid there.
