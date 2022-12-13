---
Title: Touch screen on multi-headed setup
Description: Touch screen on multi-headed setup in Debian Bullseye with XFCE
Template: withsubmenu
---

# Touch screen on multi-headed setup

This is a quick tutorial on mapping touch screens to proper physical displays. I'm running Debian Bullseye with XFCE on top at the time of writing this, but these instructions should work on many different Linux systems as well.

## List your displays

List your displays with `$ xrandr`. Example output:

```bash
~ $ xrandr
Screen 0: minimum 8 x 8, current 2560 x 2520, maximum 16384 x 16384
DVI-I-0 disconnected (normal left inverted right x axis y axis)
DVI-I-1 disconnected (normal left inverted right x axis y axis)
HDMI-0 connected 1920x1080+640+0 inverted (normal left inverted right x axis y axis) 280mm x 190mm
   2560x1440     59.95 +
   1920x1080     60.00*   60.00    59.94    50.00    60.05    60.00    50.04    50.04
   1440x900      59.89
   1400x1050     59.98
   1280x1024     60.02
   1280x960      60.00
   1280x720      60.00    59.94    50.00
   1024x768      60.00
   800x600       60.32    56.25
   640x480       59.94
DP-0 connected primary 2560x1440+0+1080 (normal left inverted right x axis y axis) 597mm x 336mm
   2560x1440     59.95*+
   1920x1080     60.00    59.94    50.00
   1680x1050     59.95
   1600x900      60.00
   1440x900      59.89
   1280x1024     75.02    60.02
   1280x800      59.81
   1280x720      60.00    59.94    50.00
   1152x864      75.00
   1024x768      75.03    70.07    60.00
   800x600       75.00    72.19    60.32    56.25
   720x576       50.00
   720x480       59.94
   640x480       75.00    72.81    59.94
DP-1 disconnected (normal left inverted right x axis y axis)
DP-2 disconnected (normal left inverted right x axis y axis)
DP-3 disconnected (normal left inverted right x axis y axis)
DP-4 disconnected (normal left inverted right x axis y axis)
DP-5 disconnected (normal left inverted right x axis y axis)
```

## Find the touch screen name

The touch display is connected to the HDMI-0 output so that's where we want to map the actual touch input also. Next, list the touch devices with `$ xinput`:

```bash
~ $ xinput
⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer              	id=4	[slave  pointer  (2)]
⎜   ↳ Logitech MX Ergo                        	id=10	[slave  pointer  (2)]
⎜   ↳ Logitech MX Ergo                        	id=11	[slave  pointer  (2)]
⎜   ↳ USB-HID Keyboard Mouse                  	id=15	[slave  pointer  (2)]
⎜   ↳ Silicon Works Multi-touch SW4101C Mouse 	id=19	[slave  pointer  (2)]
⎜   ↳ Silicon Works Multi-touch SW4101C       	id=20	[slave  pointer  (2)]
⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
    ↳ Virtual core XTEST keyboard             	id=5	[slave  keyboard (3)]
    ↳ Power Button                            	id=6	[slave  keyboard (3)]
    ↳ Power Button                            	id=7	[slave  keyboard (3)]
    ↳ Sleep Button                            	id=8	[slave  keyboard (3)]
    ↳ FiiO DigiHug USB Audio                  	id=9	[slave  keyboard (3)]
    ↳ USB-HID Keyboard                        	id=12	[slave  keyboard (3)]
    ↳ USB-HID Keyboard System Control         	id=13	[slave  keyboard (3)]
    ↳ USB-HID Keyboard Consumer Control       	id=14	[slave  keyboard (3)]
    ↳ USB-HID Keyboard                        	id=16	[slave  keyboard (3)]
    ↳ Logitech MX Ergo                        	id=17	[slave  keyboard (3)]
    ↳ Logitech MX Ergo                        	id=18	[slave  keyboard (3)]
```

## Mapping the touch input to proper display

The `Silicon Works Multi-touch SW4101C` (and the "Mouse"-variant too) is the actual touch interface. With that name and the name of the actual video output we can map the touch interface properly:

```bash
~ $ xinput --map-to-output 'Silicon Works Multi-touch SW4101C' 'HDMI-0'
```

## Surviving reboot

Even though the mapping is now correct it will not survive disconnecting the display or rebooting the machine.

To automate mapping at boot we would need to add the commands to e.g. XFCE AutoStart (Settings Manager > Session and Startup > Application Autostart). That is simple, just "add application" from the plus sign and add the following command: `bash -c "sleep 10 && xinput --map-to-output 'Silicon Works Multi-touch SW4101C' HDMI-0"`. Change the names to match your environment of course, ensure that the command gets triggered on login, and give a meaningful name to it also. Now logout and log back in to check if it works.

The mappings will still not survive disconnecting the display, but that is not something I need to consider in my setup currently. If that's something you need to consider [the ArchWiki has excellent info](https://wiki.archlinux.org/title/touchscreen#Using_a_touchscreen_in_a_multi-head_setup) on that. The ArchWiki is also what I used as a source for this tutorial, so many thanks to them!

And that's it. That will map the touch interface to the actual monitor and at least in my system also rotates the touch input by 180 degrees since the display is also rotated (in the normal XFCE settings).
