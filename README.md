# Opensource toolchain tutorial for espressif ESP32

ESP32 is a series of low-cost, low-power system on a chip microcontrollers with integrated Wi-Fi and dual-mode Bluetooth. The ESP32 series employs either a Tensilica Xtensa LX6 microprocessor in both dual-core and single-core variations, Xtensa LX7 dual-core microprocessor or a single-core RISC-V microprocessor and includes built-in antenna switches, RF balun, power amplifier, low-noise receive amplifier, filters, and power-management modules. ESP32 is created and developed by Espressif Systems, a Shanghai-based Chinese company, and is manufactured by TSMC using their 40 nm process. It is a successor to the ESP8266 microcontroller. 

Since the release of the original ESP32, a number of variants have been introduced and announced, includes ESP32, ESP32S2 / S3, and RISC-V series ESP32C3 / C6 / C5 / H2 / P4.

ESP32 have very good official tutorials at https://docs.espressif.com/projects/esp-idf/en/latest/esp32/.

Please refer to "[Standard Toolchain Setup for Linux and macOS](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html)" for Linux toolchain setup.

Since official documents is good enough, this tutorial is only a brief note for Linux.

# Hardware prerequiest
- A ESP32 devboard, S3 is recommended when this tutorial written, C3 is the RISC-V version.
- Standard JTAG debugger, Such as FTDI based.
  + NOTE: ESP32 doesn't support SWD interface

# Toolchain overview
- Compiler: xtensa (ARM) GNU Toolchain for ESP32 and ESP32S series, RISC-V GNU Toolchain for ESP32C series.
- SDK: ESP-IDF
- Programming tool: esptool.py
- Debugging: ESP32 OpenOCD / gdb

# Compiler
It's not necessary to build the xtensa or RISC-V toolchain for various ESP32 chips, esp-idf already have toolchain integrated and will download it when setup the esp idf environment. you even have no need to care about where the toolchains installed, what you only need to know is the triplet of xtensa toolchain usually is **`xtensa-esp32xx-elf-`**, `esp32xx` means `esp32`, `esp32s2` and `esp32s3`, and the triplet of RISC-V toolchain usually is **`riscv32-esp-elf-`**.

These triplets will be used when you launch gdb to debug your program.

# SDK

ESP-IDF, the *Espressif Iot Development Framework* is the official standard SDK for ESP32 series. There are a lot of development frameworks for ESP32, I will focus on esp-idf in this tutorial.

## get ESP-IDF

```
mkdir -p ~/esp && cd ~/esp
git clone --recursive https://github.com/espressif/esp-idf.git
```

## Install toolchains, various tools and python modules for ESP-IDF

After git clone finished, before you start using ESP-IDF, you need to setup toolchains and various other tools for it. ESP-IDF provide a helper script to make things easier

```
cd ~/esp/esp-idf
./install.sh esp32
```

It will download all esp32 toolchains and various other tools from github, and install them inside the user home directory: `$HOME/.espressif` on Linux.

If you prefer the Espressif download server when installing tools, you can use:

```
cd ~/esp/esp-idf
export IDF_GITHUB_ASSETS="dl.espressif.com/github_assets"
./install.sh esp32
```

After download and setup finished, you may found the `xtensa-esp32s3-elf-gcc` is not on your PATH. you need setup env vars first by:
```
. $HOME/esp/esp-idf/export.sh
```

If you plan to use esp-idf frequently, please append it to `~/.bashrc`.

## Build blink demo

There are a lot of examples in `esp-idf/examples`, using blink as example:

```
cp -r ~/esp/esp-idf/examples/get-started/blink ~
cd ~/blink
```

To build it with esp32s3 :

```
idf.py set-target esp32s3
idf.py build
```

If you open `main/blink_example_main.c`, you may be curious about where the CONFIG_BLINK_GPIO defined for `#define BLINK_GPIO CONFIG_BLINK_GPIO`, it is defined in `sdkconfig` at current dir. you can run `idf.py menuconfig` to modify this `sdkconfig` file.

After built finished, there are various firmwares generated at `build` dir, it's not necessary to care about it for now, since the programming tool will handle them automatically.


# Programming

The upstream programming tool is `esptool.py`, it is already installed when we setup toolchains for esp-idf.

Connect typec port of ESP32S3 devboard to PC USB port, usually there is one or two typec port on ESP32S3 devboard, one is for UART chip, and another one is directly connect to ESP32S3, both should work for programming.

To programming the firmwares to target ESP32 device:
```
idf.py flash
```

After programming successfully, the WS2812 RGB LED on ESP32S3 devboard will blink. A lot of variant devboards may have no LED on board, please check the schematic of your devboard.




