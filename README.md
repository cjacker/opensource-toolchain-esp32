# Opensource toolchain tutorial for Espressif ESP32

For classic ESP8266 and Ai-Thinker ESP-1S, please refer to https://github.com/cjacker/opensource-toolchain-esp8266, and ESP32-C2/C3/C6 is recommended to use instead to replace ESP8266.

ESP32 is a series of low-cost, low-power system on a chip microcontrollers with integrated Wi-Fi and dual-mode Bluetooth. The ESP32 series employs either a Tensilica Xtensa LX6 microprocessor in both dual-core and single-core variations, Xtensa LX7 dual-core microprocessor or a single-core RISC-V microprocessor and includes built-in antenna switches, RF balun, power amplifier, low-noise receive amplifier, filters, and power-management modules. ESP32 is created and developed by Espressif Systems, a Shanghai-based Chinese company, and is manufactured by TSMC using their 40 nm process. It is a successor to the ESP8266 microcontroller. 

Since the release of the original ESP32, a number of variants have been introduced and announced, includes Xtensa series ESP32, ESP32 S2 / S3, and RISC-V series ESP32 C2 / C3 / C6 / C5 / H2 / P4, etc. and we can assert that it is moving to RISC-V.

ESP32 have very good official tutorials at https://docs.espressif.com/projects/esp-idf/en/latest/esp32/.

Please refer to "[Standard Toolchain Setup for Linux and macOS](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html)" for Linux toolchain setup.

Since official documents is good enough, this tutorial is only a brief note for Linux.

# Table of contents
- [Hardware prerequiest](#hardware-prerequiest)
- [Toolchain overview](#toolchain-overview)
- [Compiler](#compiler)
- [SDK](#sdk)
  + [get ESP-IDF](#get-esp-idf)
  + [Install toolchains and tools](#install-toolchains-and-tools)
  + [Build blink demo](#build-blink-demo)
- [Programming](#programming)
- [Debugging](#debugging)
  + [Install esp32-openocd](#install-esp32-openocd)
  + [JTAG pinout of ESP32 / S2 / C2](#jtag-pinout)
  + [Launch OpenOCD](#launch-openocd)
  + [Debug](#debug)

# Hardware prerequiest
- A ESP32 devboard
  + I will use ESP32, S2, S3 and C2 / C3 / C6 in this tutorial.
  + S3 and C3/C6 is the latest model and recommended when this tutorial written. S3 is the xtensa version and C3 / C6 is the RISC-V version.
- JTAG debugger for ESP32 and ESP32 S2, such as FTDI JTAG debugger or JLink
  + NOTE 1: ESP32 doesn't support SWD interface, you can NOT use DAP-Link with ESP32.
  + NOTE 2: Only ESP32 and ESP32 S2 / C2 require an external JTAG debugger, ESP32 S3 / C3 / C6 and above have builtin jtag debug unit.

# Toolchain overview
- Compiler: Xtensa GNU Toolchain for ESP32 and ESP32S series, RISC-V GNU Toolchain for ESP32Cx series.
- SDK: ESP-IDF, **partial open source, most libraries of ble and wifi are released in binary form**
- Programming tool: esptool.py
- Debugging: ESP32 OpenOCD / gdb

# Compiler
It's not necessary to build and install the xtensa or RISC-V toolchain for various ESP32 models, esp-idf already have toolchain integrated and will download and setup when setup the esp-idf environment. What you only need to know is the triplet of xtensa toolchain usually is **`xtensa-esp32xx-elf-`**, `esp32xx` means `esp32`, `esp32s2` and `esp32s3`, and the triplet of RISC-V toolchain usually is **`riscv32-esp-elf-`**. These triplets will be used when you launch corresponding gdb to debug your program.

# SDK

ESP-IDF, the *Espressif Iot Development Framework* is the official standard SDK for ESP32 series. There are also a lot of thirdparty SDKs for different languages such as micropython, luatos, etc, they all build upon ESP-IDF. you should learn ESP-IDF first.

## get ESP-IDF

```
mkdir -p ~/esp && cd ~/esp
git clone --recursive https://github.com/espressif/esp-idf.git
```

## Install toolchains and tools
After git clone finished, before you start using ESP-IDF, you need to setup toolchains and various other tools. ESP-IDF provide a helper script to make things easier

```
cd ~/esp/esp-idf

# remove pre-installed outdated toolchain
python tools/idf_tools.py uninstall

./install.sh esp32
```

It will download the latest esp32 toolchains and various other tools from github, and install them to `$HOME/.espressif` on Linux.

Please change `./install.sh esp32` according to your target board, for example, esp32c6 should use `./install.sh esp32c6` command.

If you prefer the Espressif download server instead of github :

```
cd ~/esp/esp-idf
export IDF_GITHUB_ASSETS="dl.espressif.com/github_assets"
./install.sh esp32
```

After download and setup finished, you need setup ENV first by:
```
. $HOME/esp/esp-idf/export.sh
```

It's better to append it to `~/.bashrc`.

## Build blink demo

There are a lot of examples in `esp-idf/examples`, using blink as example:

```
cp -r ~/esp/esp-idf/examples/get-started/blink ~
cd ~/blink
```

To build it with esp32  :

```
idf.py set-target esp32
idf.py build
```

To build it with esp32s3 :

```
idf.py set-target esp32s3
idf.py build
```

To build other targets:
```
idf.py set-target <target model name>
idf.py build
```

If you open `main/blink_example_main.c`, you may be curious about where the CONFIG_BLINK_GPIO defined for `#define BLINK_GPIO CONFIG_BLINK_GPIO`, it is defined in `sdkconfig` at current dir. you can run `idf.py menuconfig` to modify `sdkconfig` file.

After built finished, there are various firmwares generated at `build` dir, it's not necessary to care about it for now, since the programming tool will handle them automatically.


# Programming

The upstream programming tool is `esptool.py`, it is already installed when we setup toolchains for esp-idf.

Usually an ESP32S3 / C3 / C6 devboard has one or two USB port exported, one is UART, and another one is builtin JTAG debug unit, both can be used for programming, you can use `lsusb` to identify the USB ports after connecting devboard to PC USB port.

To programming the firmwares to target ESP32 device:

```
idf.py flash
```

After programming successfully, the WS2812 RGB LED on ESP32S3 devboard will blink. A lot of variant devboards may have no LED on board, please check the schematic of your devboard.

**NOTE**

I noticed some ESP32S2 devboards (such as lolin S2 mini) need to **hold the boot button down, toggle reset button and release boot button** to enter flash mode. and such a devboard also can not be reset before or after flashing, you need press reset button manually to reset. Maybe There are also some error outputs when `idf.py flash`, to get rid of these error msgs, you can try run `idf.py menuconfig`, goto `Serial flasher config` menu, and change `Before flashing` to `no reset` , and change `After flashing` to `stay in bootloader`.

# Debugging

The key software and hardware components that perform debugging of ESP32 with OpenOCD over JTAG interface is presented in the diagram below:

<img src="./jtag-debugging-overview.jpg" width="50%" />

Only ESP32 and ESP32 S2/C2 require an external JTAG debugger. ESP32 S3 / C3 and above have builtin JTAG debug unit and it's not necessary to use external JTAG debugger with them (it does support external JTAG, but need burn eFuses, and burning eFuses is an irreversible operation)

## JTAG pinout

Refer to below table to wire up Jtag debugger with esp32 or s2 or c2:

| jtag | esp32   | esp32 s2 | esp32 c2 |
|------|---------|----------|----------|
| TDO  | GPIO 15 | GPIO 40  | GPIO 7   |
| TDI  | GPIO 12 | GPIO 41  | GPIO 5   |
| TCK  | GPIO 13 | GPIO 39  | GPIO 6   |
| TMS  | GPIO 14 | GPIO 42  | GPIO 4   |


Since ESP32 S3 / C3 and above have builtin JTAG debug unit, I will not list JTAG pins of those chips and will not mention about how to use external JTAG debugger. Please refer to schematics of your devboard, use `lsusb` to figure out which port belongs to builtin JTAG and connect it to PC USB port, the `lsusb` output should look like:

```
303a:1001 Espressif USB JTAG/serial debug unit
```

## OpenOCD

Upstream openocd doesn't support debugging ESP32, you have to use Espressif forked version. 

When you setup ESP-IDF SDK, the esp32 openocd already installed in `~/.espressif/tools/openocd-esp32/<version>/openocd-esp32` dir, you can use it directly.

### For ESP32, ESP32 S2 and ESP32 C2

Since it requires external JTAG debugger, the jtag interface config file depends on which JTAG debugger you use. In this tutorial, I use FT2232D, launch esp32-openocd as:

```
openocd -f <path to>/ft2232d.cfg -f <find it from esp32-openocd installation dir>/esp32.cfg
```

Change `esp32.cfg` to `esp32s2.cfg` or `esp32c2.cfg` for s2 or c2 devboard.

### For ESP32 S3 / C3 and above

ESP32 S3/C3/C6 and above have builting JTAG debug unit, launch esp32 openocd as:

```
idf.py openocd
```

The openocd output of ESP32 C6 looks like:
```
Executing action: openocd
Note: OpenOCD cfg not found (via env variable OPENOCD_COMMANDS nor as a --openocd-commands argument)
OpenOCD arguments default to: "-f board/esp32c6-builtin.cfg"
OpenOCD started as a background task 1906838
Executing action: post_debug
Open On-Chip Debugger v0.12.0-esp32-20241016 (2024-10-16-14:17)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : only one transport option; autoselecting 'jtag'
Info : esp_usb_jtag: VID set to 0x303a and PID to 0x1001
Info : esp_usb_jtag: capabilities descriptor set to 0x2000
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : esp_usb_jtag: serial (40:4C:CA:40:7B:B8)
Info : esp_usb_jtag: Device found. Base speed 24000KHz, div range 1 to 255
Info : clock speed 24000 kHz
Info : JTAG tap: esp32c6.tap0 tap/device found: 0x0000dc25 (mfg: 0x612 (Espressif Systems), part: 0x000d, ver: 0x0)
Info : [esp32c6] datacount=2 progbufsize=16
Info : [esp32c6] Examined RISC-V core; found 2 harts
Info : [esp32c6]  XLEN=32, misa=0x40903105
Info : [esp32c6] Examination succeed
Info : [esp32c6] starting gdb server on 3333
Info : Listening on port 3333 for gdb connections
```


## Debug

Launch another terminal:

```
cd blink
idf.py gdb
```

When `(gdb)` prompt showed:
```
(gdb) target remote :3333
Remote debugging using :3333
0x40000400 in ?? ()
(gdb) break blink_led
Breakpoint 2 at 0x42007d0c: file /home/cjacker/esp/blink/main/blink_example_main.c, line 31.
(gdb) c
(gdb) c
Continuing.
[esp32s3.cpu1] Debug controller was reset.
[esp32s3.cpu1] Core was reset.
[esp32s3.cpu1] Target halted, PC=0x40041A76, debug_reason=00000000
[esp32s3.cpu1] Reset cause (3) - (Software core reset)
[esp32s3.cpu0] Target halted, PC=0x42007D70, debug_reason=00000001
Set GDB target to 'esp32s3.cpu0'
[esp32s3.cpu1] Target halted, PC=0x403791EE, debug_reason=00000000
[New Thread 1070174352]
[New Thread 1070176236]
[New Thread 1070178120]
[New Thread 1070165400]
[New Thread 1070169844]
[New Thread 1070163772]
[Switching to Thread 1070174352]

Thread 2 "main" hit Breakpoint 1, app_main () at blink_example_main.c:86
86              blink_led();
(gdb)
```


