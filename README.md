# ESP32 Dog Door Control System

## Overview

This project implements an automated dog door control system using an ESP32 microcontroller and ESPHome. The system includes features such as BLE tag detection, door state sensing, motor control for opening/closing the door, and battery monitoring.

## Features

- BLE tag detection for dog presence
- Door state sensing
- Motor control for opening and closing the door
- Battery voltage and percentage monitoring
- Integration with Home Assistant
- Over-the-Air (OTA) updates
- Web server for local control
- Safety switch to prevent door opening

## Hardware Requirements

- ESP32 development board (esp32dev)
- Motor driver compatible with GPIO12, GPIO13, and GPIO14
- Door state sensor connected to GPIO5
- Voltage divider for battery monitoring connected to GPIO35
- BLE tag for the dog

## Software Requirements

- ESPHome (minimum version 2024.6.0)
- Home Assistant for full integration

## Configuration

The main configuration is defined in the YAML file. Key components include:

1. **Wi-Fi Setup**: Configure your Wi-Fi credentials in the `wifi` section.
2. **BLE Tracking**: Set up to detect a specific BLE tag (dog tag).
3. **Sensors**:
   - Door state sensor
   - Battery voltage sensor
   - Battery percentage calculation
   - BLE RSSI sensor
4. **Motor Control**: Configured for bi-directional control.
5. **Cover Component**: Implements the dog door as a cover in Home Assistant.
6. **Safety Switch**: Prevents door opening when activated.

## Installation

1. Install ESPHome on your development machine.
2. Copy the YAML configuration to a new file (e.g., `dog_door.yaml`).
3. Customize the Wi-Fi credentials and any other specific settings.
4. Compile and upload the firmware to your ESP32 board:

```esphome logs dog_door.yaml```

## Usage

Once installed and configured:

1. The system will automatically detect the dog's presence via BLE.
2. The door can be controlled manually through Home Assistant or the web interface.
3. Battery status can be monitored in Home Assistant.
4. The safety switch can be toggled to prevent automatic opening.

## Customization

- Adjust the BLE MAC address to match your dog's tag.
- Modify GPIO pins if your wiring differs from the default configuration.
- Adjust motor control parameters as needed for your specific setup.

## Troubleshooting

- Ensure all connections are correct and secure.
- Check the ESPHome logs for any error messages.
- Verify that the BLE tag is within range and broadcasting.

## Future Planned
1. Pairing with the Aqara presence sensor
2. ~~Home Assistant Automations~~ Added to Automations folder

## Parts List
1. [PetSafe Dogdoor](https://www.amazon.com/dp/B000WJ0IGA) 
2. [L298N Motor DC Stepper Motor Driver](https://www.amazon.com/dp/B0CLYBPGP9?psc=1)
3. [ESP-WROOM-32 Development Board](https://www.amazon.com/dp/B08246MCL5?psc=1)
4. [Blue Charm Beacon](https://www.amazon.com/dp/B0D5C5R76T?psc=1)
5. [20K ohm Resistor](https://www.amazon.com/dp/B07QDXVRJ8?psc=1)
6. [10K ohm Resistor](https://www.amazon.com/dp/B08QRJZ82J?psc=1)
7. [MT3608 DC-DC Step Up Boost Power Converter set to 5v](https://www.amazon.com/gp/product/B089JYBF25/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8)
8. [TP4056 Type-c USB 5V 1A Battery Charger Module](https://www.amazon.com/gp/product/B0B3Z5VV3N/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)
9. Motor and Switch from the dog door


## Contributing
Contributions are welcome!

## License
This project is licensed under the Creative Commons Attribution-NonCommercial 4.0 International License. See the [LICENSE](LICENSE) file for details.
