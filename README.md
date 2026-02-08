# ESP32-C6 Matter over Thread Router/Repeater

Pre-compiled firmware to turn an ESP32-C6-DevKitC into a **Matter over Thread router** that extends your Thread mesh network.

## What is this?

This firmware transforms your ESP32-C6 into a mains-powered Thread router that:
- Extends the range of your Thread network
- Acts as a relay for battery-powered Thread devices
- Integrates with Home Assistant and other Matter controllers
- Requires no WiFi configuration - uses **Thread only**

Perfect for improving connectivity to Thread devices that are far from your border router!

## Hardware Requirements

- ESP32-C6-DevKitC (or compatible ESP32-C6 board)
- USB cable for flashing and power
- **8MB flash** (most ESP32-C6 boards have this)

## Flashing Instructions

### Prerequisites

Install esptool:
```bash
pip install esptool
```

### Flash the Firmware

1. **Connect your ESP32-C6** to your computer via USB

2. **Find the serial port:**
```bash
   # Linux/Mac
   ls /dev/tty* | grep -E 'USB|ACM'
   
   # Usually /dev/ttyACM0 on Linux or /dev/cu.usbserial-* on Mac
```

3. **Erase the flash** (recommended for clean install):
```bash
   esptool.py --chip esp32c6 --port /dev/ttyACM0 erase_flash
```

4. **Flash the firmware:**
```bash
   esptool.py --chip esp32c6 --port /dev/ttyACM0 --baud 921600 \
     --before default_reset --after hard_reset write_flash \
     --flash_mode dio --flash_freq 80m --flash_size 8MB \
     0x0 bootloader.bin \
     0xc000 partition-table.bin \
     0x20000 light.bin
```

5. **Monitor the output** (optional but recommended):
```bash
   python3 -m serial.tools.minicom /dev/ttyACM0 115200
   # Or use: screen /dev/ttyACM0 115200
```

Replace `/dev/ttyACM0` with your actual port.

## Adding to Home Assistant

### Default Credentials

- **Manual Pairing Code:** `34970112332`
- **Discriminator:** `3840`
- **Setup PIN:** `20202021`

### Commissioning Steps

1. **Power on your ESP32-C6** (via USB or 5V power supply)

2. In Home Assistant:
   - Go to **Settings → Devices & Services**
   - Click **Add Integration**
   - Select **Matter**
   - Choose **Add Matter device**

3. **Enter the manual pairing code:** `34970112332`

4. Follow the prompts to complete commissioning

5. The device will join your Thread network and appear as a light device (the actual light control only controls the RGB LED on the ESP32 - the device functions as a router regardless)

### ⚠️ Important: Provisioning Multiple Devices

If you're flashing multiple ESP32-C6 boards with this firmware:

- **Commission devices ONE AT A TIME**
- Power off or move away other un-commissioned devices while setting up each one
- All devices share the same pairing code, so the controller may get confused if multiple are advertising simultaneously
- Once commissioned, each device gets a unique identity on your network

## Verifying Router Status

After commissioning, the device should show as a **"Routing end device"** or **"Router"** in Home Assistant's Thread integration.

### If it shows as "End device" instead:

1. Go to **Settings → Add-ons → Matter Server**
2. Find your ESP32-C6 device
3. Click **"Reinterview device"**
4. Wait for the reinterview to complete
5. The device should now properly show as a routing device

You can verify Thread network topology at:
**Settings → Devices & Services → Thread → View Network**

## Source & Build Information

This firmware is based on:

- **[ESP-IDF](https://github.com/espressif/esp-idf)** v5.5.2 - Espressif IoT Development Framework
- **[ESP-Matter](https://github.com/espressif/esp-matter)** - Espressif's Matter SDK
- **Example:** `esp-matter/examples/light` (configured for Thread-only operation)

### Key Configurations

- OpenThread FTD (Full Thread Device) enabled
- WiFi disabled (Thread only)
- Matter commissioning with default test credentials
- 8MB flash partition scheme for larger firmware

### Building from Source

If you want to build this firmware yourself:
```bash
# Clone and setup ESP-Matter
git clone --recursive https://github.com/espressif/esp-matter.git
cd esp-matter
./install.sh
. ./export.sh

# Configure and build
cd examples/light
idf.py set-target esp32c6
idf.py menuconfig  # Disable WiFi, enable OpenThread
idf.py build

# Binaries will be in: build/bootloader/bootloader.bin, 
# build/partition_table/partition-table.bin, build/light.bin
```

See the [ESP-Matter documentation](https://docs.espressif.com/projects/esp-matter/en/latest/) for detailed build instructions.

## Troubleshooting

### Device not commissioning
- Make sure only one un-commissioned device is powered on
- Check that your Thread border router is operational
- Verify the device is visible via BLE (should advertise for pairing)

### Device not extending network range
- Verify it shows as "Router" or "Routing end device" (try reinterviewing if not)
- Check Thread network topology in Home Assistant
- Ensure the device is positioned within range of existing Thread infrastructure


## License

This firmware uses components from ESP-IDF and ESP-Matter, both licensed under Apache 2.0.

## Contributing

Found an issue? Have improvements? Please open an issue or pull request!

## Disclaimer

This firmware uses **test/development credentials** (discriminator 3840, PIN 20202021). For production use, you should provision unique credentials per device. This is suitable for home/hobbyist use.
