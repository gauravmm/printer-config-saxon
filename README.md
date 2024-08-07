# Saxon

This configuration is for a Voron 2.4 printer, with the following mods:

- Aluminum parts for AB axes
- Recirculating air filter
- Case lighting
- Bed fans
- Klicky Probe

## Setup instructions

Once the hardware is all assembled, ensure that the electronics are wired as follows:

1. Electronics Setup
   1. Toolhead board (Mellow FLY SHT-42) connected via CAN to a CANable, which is connected by USB to the Raspberry Pi.
   2. Main board (BTT Octopus Pro v1) connected via USB to the Raspberry Pi
   3. Probe connected to main board, not the toolhead.

2. Software Setup
   1. Install the standard OS on a Raspberry Pi,
   2. Install [kiauh](https://github.com/dw-0/kiauh).
   3. Use kiauh to install klipper, moonraker, and fluidd.
   4. Copy these config files to `~/printer_data/config`.
   5. Check that `fluidd.cfg` matches the latest version shipped with fluidd.

3. Flashing chips
   1. Compile firmware for the Octopus Pro v1. 
      1. From the `klipper` folder, run `make menuconfig`.
         1. Select the STM32F446 with a 32KiB bootloader.
         2. Enable "extra low-level configuration options" and select the "12MHz crystal" as clock reference.
      2. Run `make` to compile the firmware.
      3. Rename it to `firmware.bin` and download it into a microSD card. Insert it into the Octopus Pro and reset it.
      4. In `printer.cfg` update the `serial` field of `[mcu]` with path to the controller.
