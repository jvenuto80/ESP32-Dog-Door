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
  name: "dog-door"
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

# Enable logging with reduced verbosity
logger:
  level: INFO
  logs:
    esp32_ble_tracker: INFO
    ble_client: INFO

# Enable Home Assistant API
api:

# Allow Over-The-Air updates
ota:
  platform: esphome

# Allow provisioning Wi-Fi via serial
improv_serial:

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

# Sets up Bluetooth LE for provisioning
esp32_improv:
  authorizer: none

web_server:

# BLE tracker for dog tag
esp32_ble_tracker:
  scan_parameters:
    interval: 100ms
    window: 100ms
    active: true

binary_sensor:
  - platform: ble_presence
    mac_address: "DD:34:02:0A:47:D2"
    name: "Dog Tag Presence"
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
    mac_address: "DD:34:02:0A:47:D2"
    name: "Dog Tag RSSI"

  - platform: adc
    pin: GPIO35
    name: "Battery Voltage"
    id: battery_voltage
    update_interval: 600s
    attenuation: 11db
    filters:
      - multiply: 3.01538461

  - platform: template
    name: "Battery Percentage"
    lambda: |-
      float voltage = id(battery_voltage).state;
      if (voltage >= 4.2) return 100.0;
      if (voltage <= 3.0) return 0.0;
      return (voltage - 3.0) / (4.2 - 3.0) * 100.0;
    update_interval: 300s
    unit_of_measurement: "%"
    accuracy_decimals: 1
    

# Motor control
output:
  - platform: gpio
    pin: GPIO12
    id: motor_direction1
  - platform: gpio
    pin: GPIO13
    id: motor_direction2
  - platform: ledc
    pin: GPIO14
    id: motor_enable
    frequency: 2000Hz

fan:
  - platform: speed
    output: motor_enable
    name: "Dog Door Motor"
    id: dog_door_motor
    speed_count: 2

# Dog door cover
cover:
  - platform: template
    name: "Dog Door"
    device_class: gate
    open_action:
      - if:
          condition:
            switch.is_off: prevent_opening
          then:
            - output.turn_on: motor_direction1
            - output.turn_off: motor_direction2
            - fan.turn_on:
                id: dog_door_motor
                speed: 2
          else:
            - logger.log: "Door opening prevented by switch"
    close_action:
      - output.turn_off: motor_direction1
      - output.turn_on: motor_direction2
      - fan.turn_on:
          id: dog_door_motor
          speed: 2
    stop_action:
      - fan.turn_off: dog_door_motor
      - output.turn_off: motor_direction1
      - output.turn_off: motor_direction2
    lambda: |-
      if (id(prevent_opening).state && id(door_state_sensor).state) {
        return COVER_OPEN;
      } else if (id(door_state_sensor).state) {
        return COVER_OPEN;
      } else {
        return COVER_CLOSED;
      }

# Prevent door opening switch
switch:
  - platform: template
    name: "Prevent Dog Door Opening"
    id: prevent_opening
    optimistic: true
    turn_on_action:
      - logger.log: "Dog door opening prevention enabled"
    turn_off_action:
      - logger.log: "Dog door opening prevention disabled"
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
1. https://www.amazon.com/dp/B000WJ0IGA PetSafe Dogdoor
2. https://www.amazon.com/dp/B0CLYBPGP9?psc=1 L298N Motor DC Stepper Motor Driver
3. https://www.amazon.com/dp/B08246MCL5?psc=1 ESP-WROOM-32 Development Board
4. https://www.amazon.com/dp/B0D5C5R76T?psc=1 Blue Charm Beacons
5. https://www.amazon.com/dp/B07QDXVRJ8?psc=1 20K ohm Resistor
6. https://www.amazon.com/dp/B08QRJZ82J?psc=1 10K ohm Resistor
7. Motor and Switch from the dog door


## Contributing
Contributions are welcome!

## License
This project is licensed under the Creative Commons Attribution-NonCommercial 4.0 International License. See the [LICENSE](LICENSE) file for details.

