## OAMS (Open AMS) Configuration Setup Guide

### Introduction

This guide provides step-by-step instructions for setting up the configuration for the OpenAMS (OAMS).
An OpenAMS is an AMS which electronics have been switched to its mechanicals compatible with Klipper.
OAMS supports Klipper only.

### Prerequisites

- Access to the shell of the Raspberry Pi connected to your 3D printer.
- Basic understanding of command line usage.
- Access to the internet for downloading necessary files.
- Familiarity with 3D printer components and their functionalities.
- A 3D Printer
- An extruder / toolhead with a filament bump cutter (such as [Filametrix](https://github.com/sorted01/Filametrix). It is also recommended the extruder is a dual hobbed gear extruder, since loading filament requires the gears to pull on the filament and overcome the spring force of a loaded extruder.  It is critical for any extruder that the path from the PTFE tubing all the way down to the extruder gears is smooth and unimpeded. Any blockage, roughness, or constriction between the PTFE and the top of the extruder gears will cause nothing but problems with the OAMS.
- A CANBus setup is mandatory; however, the ACE FPS board can be used as a canbus bridge (USB -> CANBus) for Klipper to communicate with the ACE mainboard. Hence, there is no additional cost to setting up CANBus for the printer.  If CANBus is already installed, the ACE FPS board may be operated directly as a CANBus node (instead of a CANBus bridge)
- It is mandatory you thoroughly familiarize  yourself with Klipper's excellent documentation on [CANBUs](https://www.klipper3d.org/CANBUS.html)


### Making the AMS compatible with klipper printers

OAMS uses one (or multiple AMSs) and the following custom components:
- 1x ACE (Automatic Color Exchange) filament pressure sensor (FPS) board per printer
- 1x ACE (Automatic Color Exchange) mainboard per AMS installed
- Printed parts (STLs and CAD are provided)
- 4mm outer diameter / 2mm inner diameter PTFE tubing to reach between the FPS and the toolhead extruder (4mm ID / 3mm PTFE **must only be used in the filament path ONLY after the toolhead extruder**)
- Optionally, one might need some of [these spring](https://www.amazon.com/gp/product/B076LRMLLL) to balance the friction in the PTFE tubes when they are very long (300mm bed printers and longer, specially for printers with flying gantries such as a Voron v2.4)

## Principles of operation
It is important one understand the working principle of the AMS and how it operates in order to tune and troubleshoot any possible problems with the system. Here is an excellent [introduction](https://wiki.bambulab.com/en/x1/manual/intro-ams).

Prime amongst the components, it is the filament pressure sensor (i.e. filament buffer).  The filament pressure sensor (a hall effect sensor) and its spring loaded slide magnet (hereafter referred as FPS) is the most critical of systems to have a properly working OAMS. Without a properly working FPS, the rest of the system will simply not work.

The FPS measures the amount of *compression* in the filament being fed into the toolhead extruder, and makes part of the feedback mechanism to inform the OAMS about how fast to feed the filament to the extruder, maintaining always pressure within a specified range.  The OAMS **cannot operate without this feedback mechanism**.

The FPS also acts as a limiting switch while loading filament onto the toolhead extruder.  Once the filament hits the top of the gears of the toolhead extruder, the pressure on the system increases until the OAMS turns off the follower motor and detects that it is bottomed out. It is important to understand this concept, since if the FPS slide reaches its maximum value before reaching the top of the gears of the toolhead extruder, it will instruct the follower motor in the AMS to stop, and the loading sequence will fail since the extruder will not have the filament available to pull.

Long reverse bowden tubes (collectively referred in this text as bowden tubes) necessarily produce more friction; hence, in order for the FPS to work properly, one must choose the number of springs to use together to get the right amount of pressure in the FPS to balance the friction of the tube.  For example, empirically, a Voron v.24 350mm will need two [spring](https://www.amazon.com/gp/product/B076LRMLLL) to balance the friction from the long PTFE tube.  A RatRig 3D Printer V-Core 500mm might need as many as three springs.

### Flashing the Boards

There are two boards that will need flashing, the ACE FPS and the ACE mainboard.

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

### The conversion

Converting an AMS to an OAMS system involves replacing its mainboard (located under the plastic tray inside the AMS enclosure), and and tuning the FPS, provided you already have an extruder with a filament bump cutter.  Steps are given below, along with pictures on how to make the conversion.

### Limiting Factors

- Users have reported encountering errors related to maximum extrusion during filament changes. This error message, such as "Move exceeds maximum extrusion (2.745mm^2 vs 1.440mm^2)," may appear due to settings in the slicer or firmware configurations.
- To address this issue, users have experimented with adjusting the `max_extrude_cross_section` configuration option. For instance, one user set `max_extrude_cross_section: 100` under the `[extruder]` section. However, it's essential to verify the impact of such changes and understand their implications on print quality and printer performance.
- Another limitation relates to maximum extrusion distance. Errors may occur when attempting to extrude filament beyond certain thresholds, as indicated by the "max_extrude_only_distance" setting in the printer.cfg file. Adjustments to this setting, such as setting it to `1001` under `[extruder]`, may alleviate the issue. However, users should ensure proper tuning and calibration to prevent extrusion-related errors.

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
