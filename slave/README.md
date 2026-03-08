# ESP-Hosted-MCU Slave example

This document details the **ESP-Hosted MCU Slave Example**, demonstrating the slave firmware that provides Wi-Fi and Bluetooth connectivity to host MCUs via ESP32 series co-processors.

## Overview

The slave firmware enables host MCUs to utilize the Wi-Fi and Bluetooth capabilities of ESP32 series chips through **SDIO, SPI, or UART** interfaces. This minimal example focuses on basic connectivity; however, advanced features like Network Split and Host Power Save can be optionally configured for optimized network traffic and power management.

## Supported Co-processors and Transports

The following table summarizes the supported co-processors and transport communication buses between the slave and host. This example specifically utilizes **SDIO** as the transport and **ESP32-C6** as the slave co-processor.

| Transport Supported | SDIO | SPI Full-Duplex | SPI Half-Duplex | UART |
|---|:---:|:---:|:---:|:---:|
| **Co-Processors Supported** | | | | |
| ESP32 | ✓ | ✓ | × | ✓ |
| ESP32-C2 | × | ✓ | ✓ | ✓ |
| ESP32-C3 | × | ✓ | ✓ | ✓ |
| ESP32-C5 | ✓ | ✓ | ✓ | ✓ |
| ESP32-C6/C61 | ✓ | ✓ | ✓ | ✓ |
| ESP32-S2 | × | ✓ | ✓ | ✓ |
| ESP32-S3 | × | ✓ | ✓ | ✓ |


## Example Hardware Connections

This example uses the SDIO interface. The default SDIO pin connections for the ESP32-C6 slave are as follows:

### SDIO Interface (Default for ESP32-P4-Function-EV-Board)

| Signal | GPIO | Notes |
|:-------|:-----|:------------|
| CLK | 19 | Clock |
| CMD | 18 | Command |
| D0 | 20 | Data 0 |
| D1 | 21 | Data 1 |
| D2 | 22 | Data 2 |
| D3 | 23 | Data 3 |
| Reset | EN | Reset input |

For detailed SDIO hardware connection requirements, refer to the official documentation at [https://github.com/espressif/esp-hosted-mcu/blob/main/docs/sdio.md\#3-hardware-considerations](https://github.com/espressif/esp-hosted-mcu/blob/main/docs/sdio.md#3-hardware-considerations). The GPIO pins and transport can be configured as explained in later sections.


## Quick Start Guide

### 1. Obtain the Slave Example

Execute the following commands to retrieve the slave example:

```bash
idf.py create-project-from-example "espressif/esp_hosted:slave"
cd slave
```

### 2. Set Up ESP-IDF

It is presumed that ESP-IDF has already been set up. If not, please proceed with the setup using one of the following options:

#### Option 1: Installer Way

  * **Windows**

      * Install and set up ESP-IDF on Windows as documented in the [Standard Setup of Toolchain for Windows](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/windows-setup.html).
      * Use the ESP-IDF [Powershell Command Prompt](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/windows-setup.html#using-the-command-prompt) for subsequent commands.

  * **Linux or macOS**

      * For bash:
        ```bash
        bash docs/setup_esp_idf__latest_stable__linux_macos.sh
        ```
      * For fish:
        ```fish
        fish docs/setup_esp_idf__latest_stable__linux_macos.fish
        ```

#### Option 2: Manual Way

Please follow the [ESP-IDF Get Started Guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html) for manual installation.

### 3. Set Target

Set the target co-processor using the command below:

```bash
idf.py set-target esp32c6
```

> [!TIP]
> You can **customize** the target co-processor by replacing `esp32c6` with your desired ESP32 series chip.

### 4. Customizing Configuration: Transport and Features

This is optional step. By default, SDIO transport is pre-configured.

You can access the configuration menu to choose desired configuration and features using:

```bash
idf.py menuconfig
```

The default configuration tree looks like this:
```
Example Configuration
└── Bus Config in between Host and Co-processor
    └── Transport layer
        └── Select transport: SDIO/SPI-Full-Duplex/SPI-Half-Duplex/UART
    └── <Other optional features>
```


> [!TIP]
> You can optionally **customize** the transport layer (SDIO, SPI Full-Duplex, SPI Half-Duplex, or UART), their GPIOs in use and other optional features within this menu.

### 5. Build and Flash

Build and flash the firmware to your device using the commands below, replacing `<SERIAL_PORT>` with your device's serial port:

```bash
idf.py build
idf.py -p <SERIAL_PORT> flash monitor
```

> [!TIP]
> You can **customize** the serial port (`<SERIAL_PORT>`) to match your specific hardware connection.

### 6. Where Are the Compiled BIN Files? / 编译的 BIN 文件在哪里？

#### Option A – Local build (`idf.py build`)

After a successful `idf.py build` the three flashable binaries are placed inside the **`slave/build/`** directory:

| File | Flash address | Description |
|------|--------------|-------------|
| `slave/build/network_adapter.bin` | `0x10000` | Main application firmware |
| `slave/build/bootloader/bootloader.bin` | `0x0000` | Second-stage bootloader |
| `slave/build/partition_table/partition-table.bin` | `0x8000` | Partition table |
| `slave/build/flasher_args.json` | — | esptool address map |
| `slave/build/flash_project_args` | — | Shortcut args for `idf.py flash` |

Flash all three binaries at once with:

```bash
esptool.py \
  --chip esp32c6 \
  --port /dev/ttyUSB0 \
  --baud 460800 \
  write_flash \
    0x0000  slave/build/bootloader/bootloader.bin \
    0x8000  slave/build/partition_table/partition-table.bin \
    0x10000 slave/build/network_adapter.bin
```

or simply:

```bash
idf.py -p /dev/ttyUSB0 flash
```

#### Option B – GitHub Actions CI artifact

Every successful run of the **"Build ESP32-C6 Slave Firmware (SPI Full-duplex, Zephyr host)"** workflow uploads a ready-to-use ZIP artifact:

1. Open the repository on GitHub and click the **Actions** tab.
2. Select the workflow run for your commit/branch.
3. Scroll to the **Artifacts** section at the bottom of the run summary page.
4. Download **`esp32c6-spi-zephyr-firmware`** (ZIP archive, retained for 90 days).
5. Extract the ZIP – it contains all three `.bin` files plus the flash instructions.

> [!NOTE]
> If the artifact link has expired, trigger a fresh build via **Actions → Run workflow** (workflow_dispatch).


## References

- [ESP-Hosted MCU Documentation](../../README.md)
- [ESP32-P4-Function-EV-Board Setup](../../docs/esp32_p4_function_ev_board.md)
- [Transport Layer Documentation](../../docs/)
- [Troubleshooting Guide](../../docs/troubleshooting.md)
