## Debugging

### Enable Debug Output

If you are asked to send debug info or want to fix bugs, enable debugging
first in the driver and upon request send the xpadneo related part:

```bash
echo 3 | sudo tee /sys/module/hid_xpadneo/parameters/debug_level
dmesg | egrep -i 'hid|input|xpadneo' | tee xpadneo-dmesg.txt
```

where `3` is the most verbose debug level. Disable debugging by setting the
value back to `0`.

You may want to set the debug level at load time of the driver. You can do
this by applying the setting to modprobe:
```bash
echo "options hid_xpadneo debug_level=3" | sudo tee /etc/modprobe.d/99-xpadneo-bluetooth.conf
```

Now, the driver will be initialized with debug level 3 during modprobe.

Useful information can now be aquired with the commands:

  * `dmesg`: I advise you to run `dmesg -wdH` in a terminal while you connect your controller from a second terminal to get hardware information in realtime.
  * `modinfo hid_xpadneo`: get information on xpadneo as a kernel module.
  * When your gamepad is connected, get the HID report descriptor:

```bash
xxd -c20 -g1 /sys/module/hid_xpadneo/drivers/hid:xpadneo/0005:045E:*/report_descriptor | tee >(cksum)
```

### Generated Events

If you are asked to supply the events generated by xpadneo, please run the following command:
```
perl -0777 -l -ne 'print "/dev/input/$1\n" if /Name="Xbox Wireless Controller".*Handlers.*(event[0-9]+)/s' /proc/bus/input/devices | xargs evtest
```

Do whatever you think does not behave correctly (e.g. move the sticks from left to right if you think the range is wrong)
and upload the output.

### HID device descriptor (including checksum)

If we ask you to supply the device descriptor, please post the output of the following command:
```bash
xxd -c20 -g1 /sys/module/hid_xpadneo/drivers/hid:xpadneo/0005:045E:*/report_descriptor | tee >(cksum)
```

### Bluetooth Connection

Some debugging needs a deeper low level look. You can do this by running `btmon`:
```bash
sudo btmon | tee xpadneo-btmon.txt
```

Then reproduce the problem you are observing.

We probably also need some information about the dongle:

  * Run `lsusb` and pick the device number of your dongle.
  * Run `lsusb -v -s## | tee xpadneo-lsusb.txt` where `##` is the device number picked in the previous step.
