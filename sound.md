# Useful Commands

## Logs/Status Reports

`wpctl status`

`pactl list cards`

`aplay -L`

`pactl list short sink-inputs`

`pactl info`

`pactl info | grep "Default Sink"`

## Setting Volume

Replace "62" with the sink ID as listed in output of `wpctl status`
```
wpctl set-mute 62 0
wpctl set-volume 62 1.0
```

## Tests

* `paplay /usr/share/sounds/alsa/Front_Center.wav` 
* `speaker-test -c 2 -r 48000 -D default`  # test sound
* `speaker-test -D hw:2,7 -c 2 -t sine -f 440`

You may have to do 
```
systemctl --user stop pipewire pipewire-pulse
speaker-test -c 2 -r 48000 -D default  # (or whatever other version of the `speaker-test` command)
systemctl --user start pipewire pipewire-pulse
```


# No Sound

If `speaker-test -D hw:2,7 -c 2 -t sine -f 440` or `speaker-test -c 2 -r 48000 -D default` (replace the 2 and 7 in `hw:2,7` with <card>,<device> as listed in output of `aplay -L`) works but no sound in applications or in GUI settings sound test...

In bash terminal: 
`pactl list cards`

See output like: 

			Ports:
			  hdmi-output-0 ... device.product.name = "DELL S2721QS"
			    Part of profile(s): output:hdmi-stereo

			  hdmi-output-1 ... device.product.name = "DELL S2721QS"
			    Part of profile(s): output:hdmi-stereo-extra1

			  hdmi-output-2 ... device.product.name = "DELL U2415"
			    Part of profile(s): output:hdmi-stereo-extra2

			  hdmi-output-3 ... no product name shown (not available)
			  
First try HDMI/DP port 0 on S2721QS
```
pactl set-card-profile alsa_card.pci-0000_01_00.1 output:hdmi-stereo
pactl set-default-sink alsa_output.pci-0000_01_00.1.hdmi-stereo
speaker-test -c2 -t wav

```

# Test with speaker-test
$ 

# Try HDMI/DP port 1 on S2721QS
$ pactl set-card-profile alsa_card.pci-0000_01_00.1 output:hdmi-stereo-extra1
$ pactl set-default-sink alsa_output.pci-0000_01_00.1.hdmi-stereo-extra1

$ speaker-test -c2 -t wav
