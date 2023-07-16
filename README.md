# Brave

Brave is a free and open-source web browser developed by Brave Software, Inc. based on the Chromium web browser. Brave is a privacy-focused browser, which automatically blocks some advertisements and website trackers in its default settings. Users can turn on optional ads that reward them for their attention in the form of Basic Attention Tokens (BAT), which can be used as a cryptocurrency or to make payments to registered websites and content creators.

wikipedia.org/wiki/Brave_(web_browser)

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/1e/Brave_icon_app.png/800px-Brave_icon_app.png" width="60%" height="auto" alt="brave logo" />

## How to use this Makejail

```
INCLUDE options/network.makejail
INCLUDE gh+AppJail-makejails/brave

ARG ruleset=0

OPTION template=files/linux.conf
OPTION devfs_ruleset=${ruleset}
```

Where `options/network.makejail` are the options that suit your environment, for example:

```
ARG network?
ARG interface=appjail0

OPTION alias
OPTION virtualnet=${network}:${interface} default
OPTION nat
```

In the above example `appjail0` is a loopback interface, so it must first exist before creating the jail.

A generic ruleset that allows Brave to run smoothly is as follows:

```
[devfsrules_brave=13]
add include $devfsrules_hide_all
add include $devfsrules_unhide_basic
add include $devfsrules_unhide_login

# Linux devices.
add path shm unhide
add path 'shm/*' unhide

# 3D support
add path 'dri' unhide
add path 'dri/*' unhide
add path 'drm' unhide
add path 'drm/*' unhide
add path 'pci' unhide

# USB (e.g.: webcam)
add path 'usb' unhide
add path 'usb/*' unhide

# webcam
add path 'cuse*' unhide
add path 'video*' unhide
```

The `files/linux.conf` template is as follows:

```
exec.start: /bin/true
exec.stop: /bin/true
persist
```

Open a shell and run `appjail makejail` and `appjail start`:

```sh
appjail makejail -j brave -- --ruleset 13
appjail start brave
```

You can run Brave using `brave-appjail` which is already installed.

```sh
brave-appjail
```

The other way is to run Brave from the application launcher.

<p align="center">
	<img src="https://i.ibb.co/tK1GGQW/App-Launcher-Brave.png" />
</p>

### Arguments

* `brave_tag` (default: `latest`): see [#tags](#tags).
* `brave_display` (default: `:0`): Which X server to connect it to. It can be changed at run time through the `DISPLAY` environment variable, but must be set each time `brave-appjail` is run.
* `brave_pulse_server` (default: `unix:/tmp/pulse-native`): Pulseaudio socket. It can be changed at run time through the `PULSE_SERVER` environment variable, but must be set each time `brave-appjail` is run.. See [#sound](#sound).

### Sound

Brave depends on Pulseaudio, so this Makejail will install an rc script to run Pulseaudio at startup but it is not enabled. Before enabling and starting it, configure `/usr/local/etc/pulse/system.pa`:

```
load-module module-native-protocol-unix auth-anonymous=1 socket=/tmp/pulse-native
```

Now enable and start it:

```console
# sysrc dbus_enable="YES"
dbus_enable:  -> YES
# service dbus start
Starting dbus.
# sysrc pulseaudio_enable="YES"
pulseaudio_enable:  -> YES
# service pulseaudio start
Starting pulseaudio.
N: [(null)] main.c: Running in system mode, forcibly disabling SHM mode.
N: [(null)] main.c: Running in system mode, forcibly disabling exit idle time.
```

**Note**: The socket is created even if D-Bus is not started, but to avoid the message `Unable to contact D-Bus`, it is recommended.

You should have the `/tmp/pulse-native` socket.

```console
# ls -ld /tmp/pulse-native
srwxrwxrwx  1 pulse  wheel  0 Jun 29 16:23 /tmp/pulse-native=
```

## How to build the Image

Make any changes you want to your image.

```
INCLUDE options/network.makejail
INCLUDE gh+AppJail-makejails/brave --file build.makejail

ARG ruleset=0

OPTION template=files/linux.conf
OPTION devfs_ruleset=${ruleset}
```

Build the jail:

```sh
appjail makejail -j brave -- --ruleset 13 --brave_enable_webcamd 1
```

Stop and export the jail:

```
appjail stop brave
appjail image export brave
```

### Arguments

* `brave_enable_3d` (default: `1`): Add the `brave` user to the `video` group.
* `brave_enable_webcamd` (default: `0`): Create a group named `webcamd` (GID: `145`) and add the `brave` user to it.

## Tags

| Tag      | Arch    | Version | `brave_enable_3d` | `brave_enable_webcamd` |
| -------- | ------- | ------- | ----------------- | ---------------------- |
| `latest` | `amd64` | `jammy` |         1         |           1            |

## Notes

1. This Makejail uses [Ubuntu](https://github.com/AppJail-makejails/ubuntu).
2. `audio/pulseaudio` will be installed on the host.
3. `appjail-user` will be used by `brave-appjail`. See [Unprivileged users](https://github.com/DtxdF/AppJail#unprivileged-users).
4. To use the webcam you need `multimedia/webcamd` installed on the host.
5. Audio, webcam, microphone and DRM have been tested.
6. This Makejail install `brave-appjail`, `brave-appjail.desktop` and Brave's icons.
7. This Makejail install an rc script to run Pulseaudio at startup.
8. The `/tmp` directory will be mounted on the jail.
9. `sudo(8)` is used in the jail to execute commands as the `brave` user. Note that `sudo(8)` relies on the hostname, so any change must be reflected in the `/etc/hosts` of the jail. This Makejail already correctly sets a hostname.

## Acknowledgments

This Makejail was not created without the help of

* @mrclksr: [linux-browser-installer](https://github.com/mrclksr/linux-browser-installer).
* [patovm04](https://forums.freebsd.org/members/patovm04.59820): [How to install Brave on FreeBSD 13.0+](https://forums.freebsd.org/threads/linuxlator-how-to-install-brave-linux-app-on-freebsd-13-0.78879).
* [holger](https://forums.freebsd.org/members/holger.62671): [Running Google Chrome in a dedicated Linux-Jail](https://forums.freebsd.org/threads/running-google-chrome-in-a-dedicated-linux-jail.85491).
