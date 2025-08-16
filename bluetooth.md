# Useful Commands

`bluetoothctl devices`

```
sudo systemctl enable bluetooth.service
sudo systemctl start bluetooth.service
```

```
sudo bluetoothctl
power on
agent on
default-agent
pairable on
discoverable on
scan on
```

In the following, replace XXXXX:XXXXX:XXXXX:XXXXX with the device MAC address.

```
trust XXXXX:XXXXX:XXXXX:XXXXX
pair XXXXX:XXXXX:XXXXX:XXXXX
connect XXXXX:XXXXX:XXXXX:XXXXX
```

Make sure the default agent is the actual device. For instance, if you have an extender, plug and unplug to see changes in `lsusb | grep -i bluetooth`.



# General Troubleshooting

## Ensure in Bluetooth User Group:
```
sudo usermod -aG bluetooth $USER
sudo systemctl enable bluetooth
```

## Find Device ID

Use `find /sys/bus/usb/devices/ -name "power" -exec grep -H . {} \; | grep auto` or `usb-devices | grep -A 20 -i bluetooth` or

```
cd /sys/bus/usb/devices
for dev in 1-*; do echo "$dev"; grep -q 8087 $dev/idVendor 2>/dev/null && grep -q 0033 $dev/idProduct 2>/dev/null && echo "Match: $dev"; done
```

 to find device path. **Replace `1-14` from now on where needed with the right one.**

## Check Power Suspend

Ensure

> [Policy]
> 
> AutoEnable=true
> IdleTimeout=0

and 

> [General]
>
> UserspaceHID=true
> IdleTimeout=0
  
are in `/etc/bluetooth/input.conf` (`cat /etc/bluetooth/input.conf`).

Confirm power management off: 
```
sudo iwconfig wlo1 power off
sudo systemctl stop bluetooth
sudo systemctl start bluetooth
```

`cat /sys/bus/usb/devices/*/power/control | grep auto`

Make sure there aren't power timeouts.

`cat /sys/bus/usb/devices/1-14/power/control` 

Try `echo on | sudo tee /sys/bus/usb/devices/1-14/power/control` to disable autosuspend temporarily (Use `echo auto | sudo tee /sys/bus/usb/devices/1-14/power/control` to turn back on if desired.) Replace `1-14` with your actual device path (see elsewhere in this document for how to find it).

Add to `/etc/default/grub`'s `GRUB_CMDLINE_LINUX_DEFAULT` options: `btusb.enable_autosuspend=n` (separate from other options after the colon with a space).

# Example `/etc/bluetooth/input.conf`

```
# Configuration file for the input service

# This section contains options which are not specific to any
# particular interface
[General]

# Set idle timeout (in minutes) before the connection will
# be disconnect (defaults to 0 for no timeout)
IdleTimeout=0

[Policy]
IdleTimeout=0  # for mx mouse
UserspaceHID=true  # For mx mouse


# Enable HID protocol handling in userspace input profile
# Defaults to false (HIDP handled in HIDP kernel module)
#UserspaceHID=true

# Limit HID connections to bonded devices
# The HID Profile does not specify that devices must be bonded, however some
# platforms may want to make sure that input connections only come from bonded
# device connections. Several older mice have been known for not supporting
# pairing/encryption.
# Defaults to true for security.
#ClassicBondedOnly=true

# LE upgrade security
# Enables upgrades of security automatically if required.
# Defaults to true to maximize device compatibility.
#LEAutoSecurity=true
```
