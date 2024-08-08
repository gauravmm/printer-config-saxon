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
   6. Install the Auto Z Calibration plugin.
      1. `cd ~; git clone https://github.com/protoloft/klipper_z_calibration.git`
      2. `ln -sf ~/klipper_z_calibration/z_calibration.py ~/klipper/klippy/extras/z_calibration.py`
      3. `sudo systemctl restart klipper`
   7. Configure the CANable USB-CAN adapter
      1. Set up `can0` using [this guide](https://mellow-3d.github.io/fly-sht42_klipper.html).
      2. Run `ip -s -d link show can0` and verify that `bitrate 1000000` and `txqueuelen 1024` are set.
   8. Install `katapult`
      1. Run `cd ~; git clone https://github.com/Arksine/katapult`
      2. Run `sudo apt install dfu-util -y`

3. Flashing chips
   1. Compile firmware for the Octopus Pro v1.
      1. From the `klipper` folder, run `make menuconfig`.
         1. Select the STM32F446 with a 32KiB bootloader.
         2. Enable "extra low-level configuration options" and select the "12MHz crystal" as clock reference.
      2. Run `make` to compile the firmware.
      3. Rename it to `firmware.bin` and download it into a microSD card. Insert it into the Octopus Pro and reset it.
      4. In `printer.cfg` update the `serial` field of `[mcu]` with path to the controller.

   2. Compile Katapult (formerly CanBoot) for the [Mellow FLY SHT-42](https://mellow-3d.github.io/fly-sht42_canboot_can.html) **NOTE: You only need to do this once**, the first time you set up a new board. Subsequent times, you can just flash the firmware over CAN.
      1. From the `katapult` folder run `make menuconfig`.
         1. Select the STM32F072 with a 8KiB bootloader and an 8MHz crystal.
         2. Ensure the comm. interface is `CAN bus (on PA8/PA9)` and the CAN bus speed is 1000000
         3. Set application start offset to be `8 KiB`
         4. Set "Support bootloader entry on rapid double-click of reset button"
         5. "Enable Status LED" and `!PC13`
      2. Run `make` to compile the firmware.
      3. Flash it
         1. Disconnect from the CAN cable.
         2. Connect a jumper to the DFU override port.
         3. Connect to USB. Run `lsusb` and check for DFU Mode.
         4. Flash with `sudo dfu-util -a 0 -D ~/katapult/out/katapult.bin --dfuse-address x08000000:force:mass-erase:leave -d 0483:df11`
         5. Disconnect from USB, connect to CAN.
      4. Check connectivity with `~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`.
         1. Update `printer.cfg` with the UUID reported by the command.

   3. Compile firmware for the [Mellow FLY SHT-42](https://mellow-3d.github.io/fly-sht42_klipper_can.html).
      1. From the `klipper` folder, run `make menuconfig`.
         1. Select the STM32F072 with a 8KiB bootloader and an 8MHz crystal.
         2. Ensure the comm. interface is `CAN bus (on PA8/PA9)` and the CAN bus speed is 1000000
         3. Set GPIO pins to: `PB10,PB11` to run both fans.
      2. Run `make` to compile the firmware.
      3. Flash it
         1. Get the UUID using `~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`
         1. `python3 ~/katapult/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin  -u ad7502cd87f`, substituting in the UUID as necessary.

Documentation for the [FLY SHT-42](https://mellow-3d.github.io/fly-sht42_general.html).
