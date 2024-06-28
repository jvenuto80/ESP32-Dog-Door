# Dog Door Automation with ESPHome
Turn the PetSafe Electronic Pet Door to a Home Assistant smart door.
This project automates a dog door using an ESP32 microcontroller with ESPHome firmware. It leverages Bluetooth Low Energy (BLE) tracking to detect a dog tag, controls a motor to open/close the door, and integrates with Home Assistant for easy monitoring and control.

## Components

- **ESP32 Microcontroller**: Acts as the main controller for the project.
- **BLE Presence Sensor**: Tracks the presence of a Bluetooth-enabled dog tag.
- **GPIO Sensors and Switches**: Monitor the door state and control the motor for opening/closing the door.
- **Home Assistant Integration**: Allows for seamless integration with Home Assistant, enabling remote monitoring and control.

## Features

- **Bluetooth Tracking**: Uses BLE to detect the presence of a dog tag.
- **Motor Control**: Opens and closes the dog door based on the presence of the dog tag and door state sensor.
- **Web Server**: Provides a web interface for monitoring and control.
- **OTA Updates**: Allows for over-the-air firmware updates.
- **Wi-Fi Provisioning**: Supports Wi-Fi provisioning via serial and BLE.

## Setup Instructions

### Prerequisites

- ESP32 development board
- BLE-enabled dog tag
- Motor and door mechanism
- Home Assistant setup
- ESPHome installed (version 2024.6.0 or later)

### Wiring

1. **ESP32 to Motor Control**:
   - GPIO12 -> Motor Control Input

2. **Door State Sensor**:
   - GPIO5 -> Door State Sensor Input (configured as INPUT_PULLUP)

3. **Power Supply**: Ensure the ESP32 and motor have a suitable power supply.

### Configuration

Create a file named `dog_door.yaml` with the following configuration:

```yaml
substitutions:
  name: esphome-web-315e34
  friendly_name: dog-door

esphome:
  name: ${name}
  friendly_name: ${friendly_name}
  min_version: 2024.6.0
  name_add_mac_suffix: false
  project:
    name: esphome.web
    version: '1.0'

esp32:
  board: esp32dev
  framework:
    type: arduino

logger:
  level: INFO
  logs:
    esp32_ble_tracker: INFO
    ble_client: INFO

api:

ota:
  platform: esphome

improv_serial:

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

esp32_improv:
  authorizer: none

web_server:

esp32_ble_tracker:
  scan_parameters:
    interval: 100ms
    window: 100ms
    active: true

binary_sensor:
  - platform: ble_presence
    mac_address: "xx:xx:xx:xx:xx:xx"
    name: "Dog Tag Presence"
    id: dog_tag_presence

  - platform: gpio
    pin: 
      number: GPIO5
      mode: INPUT_PULLUP
    name: "Dog Door State"
    id: door_state_sensor
    device_class: door
    filters:
      - delayed_on: 50ms
      - delayed_off: 50ms

sensor:
  - platform: ble_rssi
    mac_address: "xx:xx:xx:xx:xx:xx"
    name: "Dog Tag RSSI"
    id: dog_tag_rssi
    filters:
      - sliding_window_moving_average:
          window_size: 5
          send_every: 1
      - throttle: 1s
    on_value:
      then:
        - logger.log:
            format: "Dog Tag RSSI updated: %.1f dBm"
            args: ["x"]

cover:
  - platform: template
    name: "Dog Door"
    device_class: gate
    open_action:
      - switch.turn_on: dog_door_motor
    close_action:
      - switch.turn_off: dog_door_motor
    stop_action:
      - switch.turn_off: dog_door_motor
    optimistic: false
    assumed_state: false
    lambda: |-
      if (id(door_state_sensor).state) {
        return COVER_OPEN;
      } else {
        return COVER_CLOSED;
      }

switch:
  - platform: gpio
    pin: GPIO12
    id: dog_door_motor
    internal: true

  - platform: template
    name: "Prevent Dog Door Opening"
    id: prevent_opening
    optimistic: true
```
## Deploying Configuration
1. Install ESPHome:
Follow the ESPHome installation guide.

2. Compile and Upload Firmware:

```esphome run dog_door.yaml```

Follow the prompts to upload the firmware to your ESP32 device.

## Home Assistant Integration
1. Add ESPHome Device:
In Home Assistant, navigate to Configuration > Integrations and add the ESPHome device by specifying its IP address.

2. Monitor and Control:
Use the Home Assistant dashboard to monitor the BLE presence, door state, and control the motor.

## Usage
- **Automatic Door Operation:**
The door will open when the dog tag is detected near the door and close when the tag is out of range.

- **Manual Control:**
Use the Home Assistant interface to manually control the door or override automatic operations using the "Prevent Dog Door Opening" switch.

## Troubleshooting
- **Connection Issues:**
Ensure the ESP32 is within range of the Wi-Fi network and the BLE tag.

- **Motor Control:**
Verify the wiring and ensure the motor control input is connected correctly to the ESP32 GPIO12 pin.

- **Logs:**
Check the ESPHome logs for detailed debugging information:


```esphome logs dog_door.yaml```

## Future Planned
1. Pairing with the Aqara presence sensor
2. Home Assistant Automations

## Parts List
1. https://www.amazon.com/dp/B000WJ0IGA
2. https://www.amazon.com/dp/B0CLYBPGP9?psc=1
3. https://www.amazon.com/dp/B08246MCL5?psc=1
4. https://www.amazon.com/dp/B0D5C5R76T?psc=1
5. Motor and Switch from the dog door


## Contributing
Contributions are welcome!

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

