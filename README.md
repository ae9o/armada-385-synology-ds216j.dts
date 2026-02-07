# armada-385-synology-ds216j.dts

## Device Tree file for Synology DS216j NAS

This Device Tree allows you to install a non-stock OS, such as Debian, on
Synology DS216j NAS and get a fully functional device that will not be limited
by the manufacturer's software support period.

What works:
 - All SATA ports.
 - All USB ports.
 - Gigabit Ethernet.
 - Power and Reset buttons.
 - Fan PWM.
 - All disks' green and amber LEDs are fully configurable from the OS via GPIO.
   It took a while to find the cause of the green LEDs not working in the
   original Device Tree files!
 - The LAN LED is also fully configurable. It even blinks just like in the stock
   Synology DSM OS.
 - Brightness of all LEDs can be adjusted.

Visit the [Linux Device Hacking forum](https://forum.doozan.com/read.php?2,32146)
for detailed instructions and help on installing Debian on your device.

## Important notes

### HDD LEDs

To turn the LEDs on/off based on the [hd-idle service's](https://github.com/adelolmo/hd-idle)
actions, you will need the additional [diskstation-ledd service](https://github.com/ae9o/diskstation-ledd).

This Device Tree reproduces the Synology's default behavior. The LEDs turn on
when the disks are powered on at boot. They then blink during read/write
operations. When the [hd-idle service](https://github.com/adelolmo/hd-idle) puts
the disks into standby mode, the LEDs turn off. When the disks resume from
standby, the LEDs turn on again.

The original Device Tree files had incorrect pin assignments for UART1, AHCI0,
and the LEDs. If you're interested in specific details, they can be found in the
comments of the [DTS file](armada-385-synology-ds216j.dts).

### LAN LED

The `netdev` trigger for this LED requires additional configuration after the
NAS boots. Add these lines to the `/etc/rc.local` file:

```bash
LED="/sys/class/leds/f1072004.mdio-mii:01:green:lan"
echo 1 > $LED/link
echo 1 > $LED/tx
echo 1 > $LED/rx

# WARNING! For older distros, you may need to use "eth0" instead of "end0" as the device name.
echo end0 > $LED/device_name
```

The LAN LED is controlled directly from the Ethernet PHY. In this case, all
parameters must be specified in the MDIO node of the Device Tree, not in the
GPIO-LEDS. For this reason, the original Device Trees were unable to get this
LED to work. For more information, see the [ethernet-phy.yaml](https://www.kernel.org/doc/Documentation/devicetree/bindings/net/ethernet-phy.yaml)
file from the kernel documentation.

### LED brightness control

This NAS model uses single-channel digital potentiometer TI TPL0401A to control
LED brightness. To adjust the brightness, use the following command:

```bash
# Use a value from the range [0..127] (lower is brighter).
echo 40 > /sys/bus/iio/devices/iio:device0/out_resistance0_raw
```

Although any value can be selected from the `[0..127]` range in increments of
`1`, only certain values ​​actually change the LED brightness. For example: `0`,
`39`,`40`, `127`.

### Chassis buttons

You will need the additional [diskstation-keyd service](https://github.com/ae9o/diskstation-keyd)
to register chassis button presses.

The Power and Reset buttons are not directly accessible, so they cannot be
listed in the Device Tree. These button presses are processed by the NAS's
internal hardware. However, when these buttons are pressed, special messages are
transmitted to UART1.

In the original Synology DSM OS, a special daemon, `scemd`, is responsible for
processing these messages. The source code for this daemon is not distributed
under the GPL. However, the functionality of this daemon can easily be
replicated independently. You just need to listen for and process the messages
coming to the `/dev/ttyS1`.
