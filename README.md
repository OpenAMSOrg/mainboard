## OAMS (Open AMS) Configuration Setup Guide

### Introduction

This guide provides step-by-step instructions for setting up the configuration for the OpenAMS custom 3D printer multi-material system named OAMS (Open AMS). OAMS utilizes Klipper firmware for its operation.

### Prerequisites

- Access to the Raspberry Pi connected to your 3D printer.
- Basic understanding of command line usage.
- Access to the internet for downloading necessary files.
- Familiarity with 3D printer components and their functionalities.

### Flashing the Boards

#### Setting up the FPS Board

1. Without the ACE-FPS board being powered, place a 2.54mm jumper across the boot jumper.
2. Connect the USB-C cable from the board to the Raspberry Pi.
3. Verify the board is in DFU mode by issuing the following command in the command line: `lsusb`.
4. Clone the Katapult repository and navigate to the directory:
    ```
    sudo apt -y install git
    git clone https://github.com/Arksine/katapult
    cd katapult
    ```
5. Configure the bootloader for the FPS board:
    ```
    make menuconfig
    (Hit Q, then Save)
    make clean && make
    ```
6. Install dfu-util and flash the bootloader:
    ```
    sudo apt -y install dfu-util
    dfu-util -d 0483:df11 -a 0 -R -D ~/katapult/out/katapult.bin -s0x08000000:mass-erase:force
    ```
7. Remove the BOOT jumper from the physical board and disconnect and reconnect the USB cable.

#### Setting up the AMS Board

1. Connect the USB-C cable from the AMS board to the Raspberry Pi.
2. Repeat steps 3 to 7 mentioned above for the FPS board but using the appropriate filenames and directories.

### Configuring Klipper

1. Stop Klipper service:
    ```
    sudo service klipper stop
    sudo service moonraker stop
    ```
2. Clone the Klipper and Moonraker repositories:
    ```
    cd ~/
    mv klipper klipper-original
    git clone https://github.com/OpenAMSOrg/klipper
    mv moonraker moonraker-original
    git clone https://github.com/OpenAMSOrg/moonraker
    ```
3. Activate the Klippy virtual environment:
    ```
    source ~/klippy-env/bin/activate
    ```
4. Start Klipper and Moonraker services:
    ```
    sudo service klipper start
    sudo service moonraker start
    ```

### Setting up CANBus

1. Enable the can0 interface in the Raspberry Pi's network configuration:
    ```
    sudo nano -w /etc/network/interfaces.d/can0
    ```
2. Enter the following configuration:
    ```
    allow-hotplug can0
    iface can0 can static
        bitrate 1000000
        up ifconfig $IFACE txqueuelen 1024
    ```
3. Unplug and replug the FPS board.
4. Check for the presence of the can0 interface:
    ```
    sudo ifconfig
    ```

### Finding UUIDs for FPS and AMS Boards

1. Stop Klipper service:
    ```
    sudo service klipper stop
    ```
2. Find the UUID for the FPS board:
    ```
    ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
    ```
3. Find the UUID for the AMS board by plugging it in and repeating step 2.

### Updating Klipper Code (Optional)

If you've already installed the Klipper code, update it with the latest version from the OpenAMS repository:
```
cd ~/klipper
git pull
```

### Additional Resources

- [OpenAMS Klipper Configuration](https://github.com/OpenAMSOrg/mainboard)
- [Instructions for Setting up FPS as a CANBus Bridge](https://www.klipper3d.org/CANBUS.html)

Follow these instructions carefully to ensure the successful setup of the OAMS configuration for your 3D printer. For any further assistance or troubleshooting, refer to the provided resources or reach out to the OpenAMS community.


# OAMS (Open AMS) Startup Checks Guide

This guide will walk you through the steps to set up the startup checks for the Open AMS (OAMS) custom 3D printer multi-material system.
At this stage you should already have all the code, oams.cfg, and all of the macros in this
repository installed.  From mainsail or fluidd you should be able to see both the ACE FPS and the Mainboard.

## Configuration Setup Steps:

1. **Adjust Schmitt Trigger Ranges:**
   - Determine the optimal upper and lower ranges for the Schmitt trigger based on sensor readings.
   - Example values: Trigger_lower = 0.4, Trigger_upper = 0.65.
   - Add the option `pressure_sensor_bldc_schmitt_trigger_reverse_lower: 0.4` to control BLDC unloading pressure.

2. **Restart Klipper:**
   - After adjusting trigger ranges, restart Klipper to re-read the new values.

3. **Calibrate PTFE Tube Length:**
   - Run the calibration routine `OAMS_CALIBRATE_CLICKS` to determine the number of clicks before filament reaches the toolhead.

4. **Save Configuration:**
   - Use the `SAVE_CONFIG` command to save the PTFE tube length value for motor speed adjustments.

5. **Test Spool Loading:**
   - Use `OAMS_LOAD_SPOOL` and `OAMS_UNLOAD_SPOOL` macros to test loading and unloading for all spools.

6. **Configure Macro Variables:**
   - Adjust variables such as `retract_before_cut`, `extrusion_reload_length`, `extrusion_unload_length`, and `hotend_meltzone_compensation` in macros based on your setup and preferences.

7. **Test Filament Loading:**
   - Use macros `T0`, `T1`, `T2`, and `T3` to test filament loading, unloading, and cutting functionalities.

8. **Manually Extrude Filament:**
   - Manually extrude filament to ensure it reaches the tip of the hotend for printing readiness.

9. **Adjust Slicer Settings:**
   - Ensure slicer settings only issue `T0`, `T1`, `T2`, and `T3` commands without any other special operations.
   - Enable the "prime tower" feature in the slicer (e.g., Prusa Slicer or Orca) to facilitate multi-material printing.

## Additional Notes:
- Monitor filament extrusion to ensure it reaches the hotend tip properly.
- Adjust extrusion and unloading lengths to prevent overloading motors and ensure smooth operation.
- Test and adjust settings as necessary for optimal performance.

By following these steps, you should be able to configure the OAMS multi-material system successfully for your 3D printer. If you encounter any issues or need further assistance, refer to the provided conversation transcript or seek help from the Open AMS community.
