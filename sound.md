# Check this solution FIRST

https://forums.developer.nvidia.com/t/no-option-for-audio-over-displayport-hdmi/175889/10

Also see "suspended" section below r.e. changing "auto" -- may need to re-do on kernel/driver updates

# Useful Commands

## Logs/Status Reports

`wpctl status`

`pactl list cards`

`aplay -L` (or `aplay -l` for short)

`pactl list short sinks`

`pactl list short sink-inputs`

`pactl info`

`pactl info | grep "Default Sink"`

`LANG=C pactl info | grep '^Server Name'`

`xrandr --verbose`  # to see what display connects to what port

`inxi -Aazy`

`pw-cli info Node`

`journalctl -k | grep -Ei "ALSA|HDA|sof[-]|HDMI|snd[_-]|sound|hda.codec|hda.intel"`

## Setting Volume

Replace "62" with the sink ID as listed in output of `wpctl status`
```
wpctl set-mute 62 0
wpctl set-volume 62 1.0
```

## Tests

Replace `hw:1,3` according to your actual card & device.

* `paplay /usr/share/sounds/alsa/Front_Center.wav`
* `aplay -D plughw:1,3 /usr/share/sounds/alsa/Front_Center.wav`
* `speaker-test -c 2 -r 48000 -D default`  # test sound
* `speaker-test -D hw:1,3 -c 2 -t sine -f 440`
* `sox /usr/share/sounds/alsa/Front_Center.wav -c 2 /tmp/test_stereo.wav aplay -D hw:1,3 /tmp/test_stereo.wav`

You may have to do 
```
systemctl --user stop pipewire pipewire-pulse
speaker-test -c 2 -r 48000 -D default  # (or whatever other version of the `speaker-test` command)
systemctl --user start pipewire pipewire-pulse
```

## Settings

`pavucontrol` (maybe try `pulseaudio -k` &/or `sudo alsa force-reload` first)

# No Sound

Make sure you're in the audio group (with the `groups` command); use `sudo usermod -aG audio $USER` and reboot if needed.

Make sure everything is good with video/GPU (see video.md).

Remove pulseaudio `sudo apt remove pulseaudio*`

## Check Configuration Files/Directories
Check any of these if they exist:
* `~/.config/wireplumber/main.lua.d/90-s2721qs.lua` (e.g., `props["alsa.monitor_name"] = "DELL S2721QS"` if monitor name changes)
* `~/.asoundrc` (check alignment of card and device IDs with `aplay -l`)
* `~/.config/pipewire/pipewire-pulse.conf`
* `~/.config/pipewire/media-session.d/override-s2721qs.conf`
* `/proc/asound/card2/` (replace card 2 if needed)
* `/usr/share/pipewire/pipewire.conf` and `~/.config/pipewire/`
* `/usr/share/pipewire/pipewire-pulse.conf` and `~/.config/pipewire/pipewire-pulse.conf`

## Suspended

`aplay -l` shows all suspended?
`pw-cli info Node` just has dummy?

```
mkdir -p ~/.config/pipewire
cp /usr/share/pipewire/pipewire.conf ~/.config/pipewire/
```
Add to `~/.config/pipewire/pipewire.conf`'s "context.properties" dictionary:

```
    node.suspend-on-idle                   = false
    default.clock.force-nanosleep          = false
```

Add or uncomment `pulse.idle.timeout = 0` to `~/.config/pipewire/pipewire-pulse.conf` (copy first from `/usr/share/pipewire/pipewire-pulse.conf` if needed).

Then
`systemctl --user restart pipewire pipewire-pulse`

Try changing `/lib/udev/rules.d/60-nvidia.rules` any "auto" to "on" (or other relevant files you find using `grep 10de /lib/udev/rules.d/*`)


## `speaker-test` Works but Nothing Else Does

If `speaker-test -D hw:2,7 -c 2 -t sine -f 440` or `speaker-test -c 2 -r 48000 -D default` (replace the 2 and 7 in `hw:2,7` with <card>,<device> as listed in output of `aplay -L`) works but no sound in applications or in GUI settings sound test...

See [the-spyke/pipewire.md](https://gist.github.com/the-spyke/2de98b22ff4f978ebf0650c90e82027e)

```
sudo apt install --reinstall pipewire-media-session- wireplumber
systemctl --user --now enable wireplumber.service
sudo apt install --reinstall pipewire-audio-client-libraries
sudo cp /usr/share/doc/pipewire/examples/alsa.conf.d/99-pipewire-default.conf /etc/alsa/conf.d/
sudo cp /usr/share/doc/pipewire/examples/ld.so.conf.d/pipewire-jack-x86_64-linux-gnu.conf /etc/alsa/conf.d/
sudo apt install --reinstall libldacbt-{abr,enc}2 libspa-0.2-bluetooth pulseaudio-module-bluetooth-
```
Maybe remove `/etc/alsa/conf.d/pipewire-jack-x86_64-linux-gnu.conf` if causes issues.

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
			  

```
pactl set-card-profile alsa_card.pci-0000_01_00.1 output:hdmi-stereo
pactl set-default-sink alsa_output.pci-0000_01_00.1.hdmi-stereo
speaker-test -c2 -t wav
```

Also try ...stereo-extra1 and 2.

Also `wpctl set-default 56` (replace 56 with the sink ID as listed in output of `wpctl status`)

# Example `~/.asoundrc`
```
pcm.nvhdmi {
    type plug
    slave {
        pcm "hw:2,3"   # HDMI 0
        channels 2
    }
}
pcm.nvhdmi1 {
    type plug
    slave {
        pcm "hw:2,7"   # HDMI 1
        channels 2
    }
}
pcm.nvhdmi2 {
    type plug
    slave {
        pcm "hw:2,8"   # HDMI 2
        channels 2
    }
}
pcm.nvhdmi3 {
    type plug
    slave {
        pcm "hw:2,9"   # HDMI 3
        channels 2
    }
}
pcm.!default {
    type plug
    slave.pcm "nvhdmi"
}
```

# `speaker-test` Throws Error

Make sure `~/.asoundrc` has proper card and device IDs and uses type plug (see below). Get card and device IDs from `aplay -l` (also in `pactl list cards`).

# Fixed, Then Broken on Reboot

Try some of the stuff above (e.g., setting defaults, checking profile states, checking .conf files, etc.) then...

```
systemctl --user mask pipewire pipewire-pulse wireplumber
systemctl --user mask pipewire.socket pipewire-pulse.socket
systemctl --user stop pipewire pipewire-pulse wireplumber
systemctl --user stop pipewire.socket pipewire-pulse.socket
killall pipewire wireplumber
```

Reboot

```
systemctl --user unmask pipewire pipewire-pulse wireplumber
systemctl --user unmask pipewire.socket pipewire-pulse.socket
systemctl --user start pipewire pipewire-pulse wireplumber
systemctl --user start pipewire.socket pipewire-pulse.socket
```

May throw error, but still run it:
```
sudo modprobe -r snd_hda_codec_hdmi
sudo modprobe snd_hda_codec_hdmi
```

`aplay -l`
`aplay -D plughw:1,3 /usr/share/sounds/alsa/Front_Center.wav` (replacing 1,3 as needed)

# Worst Case

Try full re-install:
```
sudo dpkg --purge --force-depends pulseaudio alsa-base alsa-utils
sudo apt install pulseaudio alsa-base alsa-utils
sudo apt --fix-broken install
sudo alsa force-reload
```
then reboot
```
sudo apt install pulseaudio-utils
```




