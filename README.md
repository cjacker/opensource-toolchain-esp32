# Opensource toolchain tutorial for espressif ESP32

ESP32 is a series of low-cost, low-power system on a chip microcontrollers with integrated Wi-Fi and dual-mode Bluetooth. The ESP32 series employs either a Tensilica Xtensa LX6 microprocessor in both dual-core and single-core variations, Xtensa LX7 dual-core microprocessor or a single-core RISC-V microprocessor and includes built-in antenna switches, RF balun, power amplifier, low-noise receive amplifier, filters, and power-management modules. ESP32 is created and developed by Espressif Systems, a Shanghai-based Chinese company, and is manufactured by TSMC using their 40 nm process. It is a successor to the ESP8266 microcontroller. 

ESP32 have very good official tutorials at https://docs.espressif.com/projects/esp-idf/en/latest/esp32/.

Please refer to "[Standard Toolchain Setup for Linux and macOS](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html)" for Linux.

This is only a brief note about how to setup esp-idf on Linux.

# Hardware prerequiest
- A ESP32 devboard, S3 is recommended when this tutorial written, C3 is the RISC-V version.
- Standard JTAG debugger, Such as FTDI based.
  + NOTE: ESP32 doesn't support SWD interface

# Toolchain overview
- Compiler: xtensa GNU Toolchain for ESP32 and ESP32S series, RISC-V GNU Toolchain for ESP32C series.
- SDK: esp-idf
- Programming tool: esptool.py
- Debugging: ESP32 OpenOCD / gdb


