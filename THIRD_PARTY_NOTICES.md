# Third-party notices

The firmware files in `flasher/bin` contain the following third-party software. Each component is licensed under its own terms.

| Component | Used for | License |
|---|---|---|
| [Arduino core for the ESP32](https://github.com/espressif/arduino-esp32) | ESP32 / S2 / S3 / C3 / C6 framework | LGPL-2.1 |
| [ESP-IDF](https://github.com/espressif/esp-idf) | ESP32 SDK (via the Arduino core) | Apache-2.0 |
| [Arduino core for the ESP8266](https://github.com/esp8266/Arduino) | ESP8266 framework | LGPL-2.1 |
| [Adafruit NeoPixel](https://github.com/adafruit/Adafruit_NeoPixel) | addressable LED driver | LGPL-3.0 |
| [ArduinoJson](https://github.com/bblanchon/ArduinoJson) | JSON parsing | MIT |

The flash page loads [ESP Web Tools](https://github.com/esphome/esp-web-tools) (Apache-2.0) from unpkg.com.

The license texts are available at the linked projects. If you need the object files to relink the firmware with a modified version of an LGPL component, contact the copyright holder through this repository.
