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
