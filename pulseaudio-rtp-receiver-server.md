---
layout: default
title: Pulseaudio RTP receiver server
description: Configure a headless Pulseaudio server that accepts connections via RTP.
tags: pulseaudio rtp systemd audio server
---

## Based on Debian Stretch 9.2 20171016

### Server starts all functions at boot, no X needed

Install pulseaudio-module-zeroconf on both client and server:

```sh
sudo apt-get install pulseaudio-module-zeroconf
```

### Server side:

Utilize systemd user services. Needed for pulseaudio to start after network is actually up, rtp module will fail without it.

Groups useful or needed for the user starting pulseaudio:

```sh
cat /etc/group | grep radiouser

audio:x:29:radiouser,pulse
systemd-journal:x:102:radiouser
systemd-timesync:x:103:radiouser
systemd-network:x:104:radiouser
radiouser:x:1000:
pulse:x:113:radiouser
pulse-access:x:114:radiouser

```

1. Pulseaudio user configuration, creating the listener:

   ```
   # ~/.config/pulse/default.pa

   .include /etc/pulse/default.pa
   # The following settings override those in above/systemwide file.

   # This startup script is used only if PulseAudio is started per-user
   # (i.e. not in system mode)

   ### Network access (may be configured with paprefs
   # default lines modified to restrict by IP address, not using cookies to authenticate
   #load-module module-esound-protocol-tcp
   #load-module module-native-protocol-tcp
   load-module module-zeroconf-publish
   load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1;172.30.0.1;172.30.0.250 auth-anonymous=1

   ### Load the RTP receiver module (also configured via paprefs, see above)
   load-module module-rtp-recv

   ```

1. Link network target (no systemctl enable needed):

   ```sh
   systemctl --user link /lib/systemd/system/network-online.target
   ```


1. Enable lingering services to make them not exit with user logout:

   ```sh
   sudo loginctl enable-linger radiouser
   ```

1. Create

   ```sh
   ~/.config/systemd/user/pulseaudio.service
   ```

   based on

   ```sh
   /usr/lib/systemd/user/pulseaudio.service
   ```

   from debian stretch.

   ```c
   [Unit]
   Description=Sound Service

   # We require pulseaudio.socket to be active before starting the daemon, because
   # while it is possible to use the service without the socket, it is not clear
   # why it would be desirable.
   #
   # A user installing pulseaudio and doing `systemctl --user start pulseaudio`
   # will not get the socket started, which might be confusing and problematic if
   # the server is to be restarted later on, as the client autospawn feature
   # might kick in. Also, a start of the socket unit will fail, adding to the
   # confusion.
   #
   # After=pulseaudio.socket is not needed, as it is already implicit in the
   # socket-service relationship, see systemd.socket(5).
   Requires=network-online.target sound.target pulseaudio.socket
   After=network-online.target sound.target

   [Service]
   # Note that notify will only work if --daemonize=no
   Type=notify
   ExecStart=/usr/bin/pulseaudio --daemonize=no --realtime --disallow-exit --no-cpu-limit
   Restart=on-failure

   [Install]
   Also=pulseaudio.socket
   WantedBy=default.target

   ```

1. Check to see if files exist:

   ```sh
   ls ~./config/systemd/user/{.,default.target.wants}

   /home/radiouser/.config/systemd/user/.:
   default.target.wants  network-online.target  pulseaudio.service  sockets.target.wants

   /home/radiouser/.config/systemd/user/default.target.wants:
   pulseaudio.service
   ```


1. Start pulseaudio:

   ```sh
   systemctl --user start pulseaudio.service
   ```


1. Check sink and source levels and mute with

   ```sh
   pacmd
   pactl
   ```

  If all is well, enable the user service:

  ```sh
  systemctl --user enable pulseaudio.service
  ```

### Client side:

Configure a source to broadcast with [Paprefs](https://freedesktop.org/software/pulseaudio/paprefs/#documentation). Make sure you have the correct IP for your machine if you used that or configure [cookie authentication](https://wiki.archlinux.org/index.php/PulseAudio/Configuration#Connection_.26_authentication).

Other programs that broadcast RTP can be used, YMMV.

