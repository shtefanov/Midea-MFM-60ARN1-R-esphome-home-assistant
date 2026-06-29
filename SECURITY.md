# Security

This project is intended to be public. Do not commit real ESPHome or Home
Assistant secrets.

Never publish:

- Wi-Fi SSID or password for a private network
- ESPHome API encryption keys
- OTA passwords
- Home Assistant long-lived access tokens
- Private Home Assistant entity IDs if they reveal your site layout
- Photos that expose serial numbers or private labels you do not want public

Use `esphome/secrets.example.yaml` as a template and keep the real
`secrets.yaml` outside Git.

## Electrical Safety

This project involves opening and wiring an air conditioner. Disconnect mains
power before working inside the unit. Use a 5V-to-3.3V level shifter between
the CN700 UART interface and ESP32 GPIO pins.
