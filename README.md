# Brave

Brave is a free and open-source web browser developed by Brave Software, Inc. based on the Chromium web browser. Brave is a privacy-focused browser, which automatically blocks some advertisements and website trackers in its default settings. Users can turn on optional ads that reward them for their attention in the form of Basic Attention Tokens (BAT), which can be used as a cryptocurrency or to make payments to registered websites and content creators.

wikipedia.org/wiki/Brave_(web_browser)

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/1e/Brave_icon_app.png/800px-Brave_icon_app.png" width="60%" height="auto" alt="brave logo" />

## How to use this Makejail

```sh
appjail makejail -j brave -f gh+AppJail-makejails/brave \
    -o x11 \
    -o template=/usr/local/share/examples/appjail/templates/linux.conf \
    -o alias \
    -o virtualnet=":appjail0 default" \
    -o nat
```

**Tip**: Read [Alias & Virtual Networks](https://appjail.readthedocs.io/en/latest/networking/virtual-networks/alias-and-virtual-networks/) to see how to create the `appjail0` interface.

You can run Brave using `brave-appjail` which is already installed.

```sh
brave-appjail
```

**Attention**: The above script assumes that you have already configured AppJail to run with trusted users. If you have not already done so, read [Trusted Users](https://appjail.readthedocs.io/en/latest/trusted-users/).

The other way is to run Brave from the application launcher.

<p align="center">
	<img src="https://i.ibb.co/tK1GGQW/App-Launcher-Brave.png" />
</p>

**Read** [#notes](#notes) to get more information about this Makejail.

### Arguments

* `brave_tag` (default: `latest`): see [#tags](#tags).
* `brave_display` (default: `:0`): Which X server to connect it to. It can be changed at run time through the `DISPLAY` environment variable, but must be set each time `brave-appjail` is run.
* `brave_enable_3d` (default: `1`): Add the `brave` user to the `video` group and enable the following devices: `dri`, `dri/*`, `drm`, `drm/*`, `pci`.
* `brave_enable_webcamd` (default: `1`): Create a group named `webcamd` (GID: `145`) and add the `brave` user to it. This option also enable the following devices: `usb`, `usb/*`, `cuse*`, `video*`.
* `brave_tz` (default: `UTC`): Default time zone used by Brave. You can change this at run time through the `TZ` environment variable, but must be set each time `brave-appjail` is run.

## How to build the Image

```sh
appjail makejail -j brave -f "gh+AppJail-makejails/brave --file build.makejail" \
    -o alias \
    -o virtualnet=":appjail0 default" \
    -o nat \
    -o template=/usr/local/share/examples/appjail/templates/linux.conf
appjail stop brave
appjail image export brave
```

## Tags

| Tag      | Arch    | Version |
| -------- | ------- | ------- |
| `latest` | `amd64` | `jammy` |

## Notes

1. This Makejail uses [Ubuntu](https://github.com/AppJail-makejails/ubuntu).
2. `audio/pulseaudio` will be installed on the host.
3. Pulseaudio is started using `brave-appjail` to create a UNIX socket (`${HOME}/.brave-pulse/pulse-native`).
4. To use the webcam you need `multimedia/webcamd` installed on the host.
5. This Makejail install `brave-appjail`, `brave-appjail.desktop` and Brave's icons.
6. `sudo(8)` is used in the jail to execute commands as the `brave` user. Note that `sudo(8)` relies on the hostname, so any change must be reflected in the `/etc/hosts` of the jail. This Makejail already correctly sets a hostname.

### Known Issues

**Audio and microphone do not work**:

The following error appears every time we try to play audio or use the microphone:

````
*** stack smashing detected ***: terminated
[1101/112318.703716:ERROR:ptracer.cc(43)] ptrace: Invalid argument (22)
[1101/112318.703893:WARNING:process_reader_linux.cc(400)] Couldn't initialize main thread.
[1101/112318.704123:ERROR:proc_task_reader.cc(46)] format error
[1101/112318.704247:WARNING:exception_snapshot_linux.cc(391)] thread ID 101288 not found in process
[1101/112318.704503:ERROR:process_snapshot_linux.cc(129)] thread not found 101288
[1101/112318.705686:ERROR:proc_task_reader.cc(46)] format error
```` 

## Acknowledgments

This Makejail was not created without the help of

* @mrclksr: [linux-browser-installer](https://github.com/mrclksr/linux-browser-installer).
* [patovm04](https://forums.freebsd.org/members/patovm04.59820): [How to install Brave on FreeBSD 13.0+](https://forums.freebsd.org/threads/linuxlator-how-to-install-brave-linux-app-on-freebsd-13-0.78879).
* [holger](https://forums.freebsd.org/members/holger.62671): [Running Google Chrome in a dedicated Linux-Jail](https://forums.freebsd.org/threads/running-google-chrome-in-a-dedicated-linux-jail.85491).
