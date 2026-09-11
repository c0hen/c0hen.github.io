---
layout: default
title: Linux audio
description: Linux audio settings and debugging on a wireplumber stack
tags: linux audio wireplumber pipewire pulseaudio alsa headphones detection hdaudio
---

* Table of contents
{:toc}

## Linux audio

### Linux headphone autodetection not working

#### Disable jack detection

Disable jack detection in alsamixer by Auto-Mute: disabled

In case auto mute does not remain disabled after a reboot:

- `/usr/bin/amixer -c 0 sset "Auto-Mute Mode" Disabled`

- `hda-verb` from alsa-tools.

#### To retask a jack, for example front jacks for extra surround channels:
- install alsa-tools-gui
- run `hdjackretask`
- tick advanced override
- create boot override, file `/etc/modprobe.d/hda-jack-retask.conf` appears
pointing to a file in `/usr/lib/firmware/hda-jack-retask.fw`

Create overrides Jack detection --> not present, possibly for both front and rear panels.

##### Retask rear mic jack as headphones

Pink mic, rear side, pin id 0x18 - retask to headphone, front channel group, advanced override checked on the right side of the screen.

If it works, "Install boot override".

### Linux audio wireplumber and pipewire debugging

Pipewire configuration is in `/usr/share/pipewire/pipewire.conf.d/`.
Configuration snippets available: `/usr/share/pipewire/pipewire.conf.avail/`.

Packages needed for alsa and pulseaudio outputs are wireplumber, pipewire-alsa, pipewire-pulse.

- Verify wireplumber / pipewire.

```sh
LANG=C pactl info | grep '^Server Name'
```
```
Server Name: PulseAudio (on PipeWire 1.4.2)
```
```sh
LANG=C aplay -L | grep -A 1 default
```
```
sysdefault
    Default Audio Device
lavrate
--
default
    Default ALSA Output (currently PipeWire Media Server)
hw:CARD=USB,DEV=0
--
sysdefault:CARD=USB
    Scarlett 2i4 USB, USB Audio
    Default Audio Device
front:CARD=USB,DEV=0
--
sysdefault:CARD=Generic
    HD-Audio Generic, ALC1220 Analog
    Default Audio Device
front:CARD=Generic,DEV=0
```

- `wpctl status`
- `wpctl inspect ID`, `wpctl inspect 35`
- `pw-dump`
- `pw-top` for audio processes
- `pactl list sinks` for pulseaudio sinks

### Pipewire audio crackling, fine tuning

The most important step to solve crackling seems to be disabling audio device suspend.

```
#~/.config/wireplumber/wireplumber.conf.d/10-alsa-vm.conf
actions = {
  update-props = {
    # disable audio device suspend, should fix crackling 20260911
    session.suspend-timeout-seconds = 0
```
Restart wireplumber and pipewire wholly to apply.
```sh
systemctl --user restart wireplumber pipewire pipewire-pulse
```

#### Add audio rates in pipewire

Some applications via wine need 44100 Hz to work properly.
```
# ~/.config/pipewire/pipewire.conf.d/10-rates.conf
# Adds more common rates
#Note that this is not enabled by default for now because of kernel driver bugs that need to be fixed/worked around first. There are also potentially some bugs with bluetooth devices and synchronization.
#Note that when switching rates, the quantum sizes are scaled relative to the default clock rate.
context.properties = {
    default.clock.allowed-rates = [ 44100 48000 88200 96000 ]
}
```

#### Further steps

Thanks to Rabcor [for their guide](https://forum.manjaro.org/t/howto-troubleshoot-crackling-in-pipewire/82442/1).

- QUANT = Quantum = Latency = Buffer Size.
Quantum divided by clock rate equals latency in seconds
(E.G. 1024/48000=0.02s or 21ms)
- Worst case scenario for below settings is setting the system’s minimum latency to ~40ms, this is not good for professional audio, but it is perfectly solid for consumer audio. In most cases though you will only have ~20ms min latency which is a lot better.
- The default max latency in PipeWire is 170ms (8192/48000), it is probably safe to reduce to 85ms (4096/48000) which is still an acceptable latency for consumer audio, but wait with that until after you solve your initial crackling problem first.
- For reference, standard audio on Windows, DirectSound normally has around 50-80ms latency. (WASAPI has around 10-30ms latency, ASIO has 1-10ms)
- You can use pw-metadata to force a system-wide clock rate or quantum setting with:
`pw-metadata -n settings 0 clock.force-rate`
`pw-metadata -n settings 0 clock.force-quantum`
(where 0 is the id of your sound card, you can see the IDs of all PipeWire detected sound cards by running the command without arguments), this can be quite useful to test different latency and frequency settings on the fly.
- If you are unsure whether your application is using pipewire or pipewire-pulse, you can combine the commands to set the sound buffer size like so:
```
PULSE_LATENCY_MSEC=83 PIPEWIRE_LATENCY=1024/48000 <COMMAND>
```

- You could set the above as a bash alias or make it a script:
```sh
#!bin/sh
PULSE_LATENCY_MSEC=83 PIPEWIRE_LATENCY=1024/48000 "$@"
```
