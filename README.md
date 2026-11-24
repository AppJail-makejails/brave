# brave

Brave is a Chromium based web browser that blocks ads and trackers by default. It ships with a built-in ad blocker, HTTPS upgrading, fingerprinting protection, a Tor enabled private window and an optional cryptocurrency wallet.

wikipedia.org/wiki/Brave_(web_browser)

<img src="https://thumb.wikimedia.org/wikipedia/commons/thumb/9/93/Brave_Logo_%282024%29.svg/1280px-Brave_Logo_%282024%29.svg.png" width="30%" height="auto" alt="brave logo">

## How to use this AppJail

### General usage

You can obtain the AppJail from the releases section of this repository. However, to go beyond a simple download and be able to update it conveniently from the console, the simplest complementary tool for our purposes is [sysutils/bin](https://freshports.org/sysutils/bin), a binary manager:

```console
$ doas pkg install -y bin
```

Install the latest version of this AppJail by running the following command:

```console
$ mkdir -p ~/bin
$ bin install https://github.com/appjail-makejails/brave
```

Or update it if it is already installed:

```console
$ bin update brave.appjail
```

Assuming `~/bin` is in your `PATH`, you can run the AppJail simply by using the following command:

```console
$ brave.appjail
```

Remember that when running an AppJail in portable mode, you must install the key used to verify the binary:

```console
$ cat << "EOF" | doas x11appjail trust dtxdf@disroot.org -
untrusted comment: dtxdf@disroot.org (x11appjail) public key
RWSZbdqRaZVSgICvhui+nrVbXbWw25jyZx/3lhaPzSmVi1Pgvk2DAB1h
EOF
$ x11appjail trusted
KEY                                                                   COMMENT
37e1a7da5478a29ec3d38ecb14919b107beab67cc0de0b473b5b018f018e1ccb.pub  dtxdf@disroot.org (x11appjail) public key
```

A system-wide installation requires only root access; the key is not necessary. However, it is strongly recommended to verify the binary before installation, making it necessary to install the key anyway.

```console
$ x11appjail verify ~/bin/brave.appjail
Signature Verified
$ doas ~/bin/brave.appjail --install
$ x11appjail run brave
```

An AppJail creates the jail only if it does not already exist or if the AppJail detects a valid change in its checksum (e.g.: after an update). This means that updates to the OCI image used by the AppJail are only checked at the creation time. If you need to update the OCI image, simply destroy the jail:

```console
$ x11appjail destroy-jail brave
```

Once you run the AppJail again, the OCI image is pulled again only if it is newer than the one on your system.

### D-Bus

To achieve better integration with the desktop environment, the host's D-Bus is shared via `xdg-dbus-proxy(1)`. By default, the system bus is not shared and the session bus is filtered. This happens transparently and you don't need to do anything. However, you can modify the behavior by creating user attributes that affect how AppJail interprets them when starting the D-Bus proxy.

```console
$ x11appjail attr put brave:default.dbus-user.talk.notifications "org.freedesktop.Notifications"
```

By defining this attribute, notifications will work as if they were running on your host. Note that this is just an example, and the application running within the jail may or may not use notifications.

Both the session bus and the system bus can be completely disabled by simply setting `<appname>:<profile>.dbus-user` or `<appname>:<profile>.dbus-system` to `none`. The system bus is usually disabled by default, as in most cases only the session bus is needed.

```console
$ x11appjail attr put brave:default.dbus-user none
```

Note that if you have already started the AppJail, in order for the changes to take effect, you must destroy the jail and then run the AppJail again.

```console
$ x11appjail destroy-jail brave
```
### Keyboard and mouse reference (Openbox)

The Openbox environment uses the following key and mouse bindings. They are
defined in Openbox's `rc.xml`.

> **Note:** `W` means the **Super** key (usually the Windows key), and `S`
> means **Shift**.

#### Keyboard bindings

| Binding | Action |
|---------|--------|
| `W-q` | Close the active window (`Close`) |
| `W-S-q` | Exit Openbox with confirmation (`Exit` with `prompt=yes`) |
| `W-f` | Toggle fullscreen (`ToggleFullscreen`) |
| `W-w` | Go to the next window (`NextWindow`) |
| `W-S-w` | Go to the previous window (`PreviousWindow`) |
| `W-l` | Show the combined client list menu (`ShowMenu > client-list-combined-menu`) |

#### Mouse bindings

| Context | Button / action | Resulting action |
|---------|-----------------|------------------|
| **Frame** (window frame) | `W-Left` (Press) | Focus and raise the window (`Focus`, `Raise`) |
| | `W-Left` (Click) | Unshade the window (`Unshade`) |
| | `W-Left` (Drag) | Move the window (`Move`) |
| | `W-Right` (Press) | Focus, raise, and unshade (`Focus`, `Raise`, `Unshade`) |
| | `W-Right` (Drag) | Resize the window (`Resize`) |
| | `W-Middle` (Press) | Lower the window to the bottom (`Lower`, `FocusToBottom`, `Unfocus`) |
| **Titlebar** | `Left` (Drag) | Move the window (`Move`) |
| | `Left` (DoubleClick) | Toggle maximize (`ToggleMaximize`) |
| | `Up` (Click) | Shade the window if it is not shaded (`Shade` conditional) |
| | `Down` (Click) | Unshade the window if it is shaded (`Unshade` conditional) |
| **Titlebar Top Right Bottom Left TLCorner TRCorner BRCorner BLCorner** | `Left` (Press) | Focus, raise, and unshade (`Focus`, `Raise`, `Unshade`) |
| | `Middle` (Press) | Lower to the bottom (`Lower`, `FocusToBottom`, `Unfocus`) |
| | `Right` (Press) | Focus, raise, and show the client menu (`ShowMenu > client-menu`) |
| **Top** (top edge) | `Left` (Drag) | Resize from the top edge (`Resize` edge `top`) |
| **Left** (left edge) | `Left` (Drag) | Resize from the left edge (`Resize` edge `left`) |
| **Right** (right edge) | `Left` (Drag) | Resize from the right edge (`Resize` edge `right`) |
| **Bottom** (bottom edge) | `Left` (Drag) | Resize from the bottom edge (`Resize` edge `bottom`) |
| | `Right` (Press) | Focus, raise, and show the client menu (`ShowMenu > client-menu`) |
| **TRCorner BRCorner TLCorner BLCorner** (corners) | `Left` (Press) | Focus, raise, and unshade (`Focus`, `Raise`, `Unshade`) |
| | `Left` (Drag) | Resize (`Resize`) |

> **Note:** `W-Left`, `W-Right`, and `W-Middle` mean that you must hold the
> **Super** key while clicking the left, right, or middle mouse button,
> respectively.
### User Permissions

By default, the jail lacks the necessary devices for sound, the webcam, hardware acceleration, etc., to work as expected. To achieve this, the system operator must grant the user certain permissions, and the user must allow the jail to be created with the specific permissions required for that session. This is implemented using attributes:

```console
$ UID=$(id -u)
$ doas x11appjail sys-attr users.${UID}.perms "enable_3d sound"
```

The previous command assumes that the shell process is running with unprivileged user rights and can use `doas(1)` to elevate privileges and execute the `sys-attr` command. This command will create the attribute based on the shell process's UID, with `enable_3d` and `sound` permissions. The user-side counterpart is as follows:

```console
$ x11appjail attr put brave:default.allow.enable_3d
$ x11appjail attr put brave:default.allow.sound
```

Although the user can create other attributes corresponding to actual permissions, such as `usb` or `webcam`, these are completely ignored if the system operator does not assign them.

If the jail has already been created, it must be destroyed for the changes to take effect.

```console
$ x11appjail destroy-jail brave
```

When you run this AppJail again, the devices will appear inside the jail, and sound and hardware acceleration will work as expected.

### Fonts

The OCI image containing all the necessary files does not install any fonts other than those essential for the application. However, this AppJail has the capability to mount system fonts in read-only mode, an operation that consumes no space.

```console
$ doas x11appjail sys-attr put mount.system-fonts
```

If the jail has already been created, it must be destroyed for the changes to take effect.

```console
$ x11appjail destroy-jail brave
```

Once the jail is recreated, the process may take some time to generate the font cache, depending on how many fonts you have on the system.

### Networking

Virtual networks are the recommended option for network connectivity. They offer granular control and allow for further isolation of the jail. However, this requires configuring and installing `appjail(1)` as specified in the "[Getting Started](https://appjail.readthedocs.io/en/latest/getting-started/)" guide.

```console
$ doas x11appjail sys-attr put network.mode virtualnet
```

If you are not concerned about the complete lack of network-level isolation, you can inherit the host's network stack.

```console
$ doas x11appjail sys-attr put network.mode inherit
```

If the jail has already been created, it must be destroyed for the changes to take effect.

```console
$ x11appjail destroy-jail brave
```

Once the jail has been recreated, you can check network connectivity by executing the following command:

```console
$ x11appjail run-cmd brave ping -c4 1.1.1.1
PING 1.1.1.1 (1.1.1.1): 56 data bytes
64 bytes from 1.1.1.1: icmp_seq=0 ttl=51 time=36.769 ms
64 bytes from 1.1.1.1: icmp_seq=1 ttl=51 time=36.617 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=51 time=44.956 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=51 time=36.360 ms

--- 1.1.1.1 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 36.360/38.675/44.956/3.629 ms
```

The above assumes that the jail is allowed to connect to `1.1.1.1` and send ICMP packets.


### Attributes
#### User Attributes

| Name | Description |
| --- | --- |
| `<appspec>.dbus-system` | Configure the system bus mode. There are three modes:<br><br>1. `filter`: Enables filtering.<br>2. `unfiltered`: Disables filtering. This mode is insecure and is recommended only for development.<br>3. `none`: Completely disables the session bus. This is the default mode unless explicitly stated otherwise.<br>|
| `<appspec>.dbus-system.<policy>.*` | Apply a policy or rule to the system bus, where `<type>` can be `own`, `call`, `talk`, `see`, and `broadcast`. See `xdg-dbus-proxy(1)` for details.|
| `<appspec>.dbus-user` | Configure the session bus mode. There are three modes:<br><br>1. `filter`: Enables filtering. This is the default mode unless explicitly stated otherwise.<br>2. `unfiltered`: Disables filtering. This mode is insecure and is recommended only for development.<br>3. `none`: Completely disables the session bus.<br>|
| `<appspec>.dbus-user.<policy>.*` | Apply a policy or rule to the session bus, where `<type>` can be `own`, `call`, `talk`, `see`, and `broadcast`. See `xdg-dbus-proxy(1)` for details.|
| `<appspec>.dpi` | Sets the resolution for all screens, in dots per inch. By default, no DPI is set.|
| `<appspec>.allow.enable_3d` | This permission will execute the application using VirtualGL and make hardware acceleration-related devices visible.|
| `<appspec>.env.<label>.*` | User-defined custom environment variables. `<label>` is an arbitrary name, as it is not part of the environment. The value of the attribute will be the environment variable with its value in the format `<name>=<value>`.|
| `<appspec>.jail.ephemeral` | Mark the jail as ephemeral. See `ephemeral` option in `appjail-quick(1)` for details.<br><br>Although the jail may be destroyed, its data is preserved in the user directory (see `${X11APPJAIL_USERDIR}` in `x11appjail-spec(5)`).<br>|
| `<appspec>.dbus-user.ignore.broadcast.portal` | If this attribute exists, `org.freedesktop.portal.*=@/org/freedesktop/portal/*` will not be applied in the `broadcast` rule.|
| `<appspec>.dbus-user.ignore.call.portal` | If this attribute exists, `org.freedesktop.portal.*=*` will not be applied in the `call` rule.|
| `<appname>:<profile>.dbus-user.ignore.own.mpris` | If this attribute exists, `org.mpris.MediaPlayer2.brave.*` will not be applied in the `own` policy.|
| `<appspec>.nofullscreen` | Prevents devilspie2 from setting windows to full screen when they can be maximized.|
| `<appspec>.screen` | Screen size. If this is not set, allow the window to be resized.|
| `<appspec>.allow.sound` | This permission will make sound-related devices visible.|
| `<appspec>.allow.usb` | This permission will make usb-related devices visible.|
| `<appspec>.virtualgl.display` | If the user has the `enable_3d` permission, this specifies the display or EGL device to be used for 3D rendering. Since using EGL is the only logical choice for this project, the default value is `egl`. In multi-GPU environments, it is possible to specify a particular device.|
| `<appspec>.allow.webcam` | This permission will make webcam-related devices visible.|

#### System Attributes

| Name | Description |
| --- | --- |
| `<appspec>.labels` | A space-separated list of label names.|
| `network.mode` | Network mode. Default is `none`.<br><br>There are three modes:<br><br>1. `virtualnet`: This option is recommended, as it provides isolation and allows a more fine-grained control. It's necessary to install and configure AppJail on the host, as specified in the "[Getting Started](https://appjail.readthedocs.io/en/latest/getting-started/)" guide.<br>2. `inherit`: This mode does not provide network isolation. From a networking perspective, it is exactly the same as running the application on the host.<br>3. `none`: Completely disable the network stack.<br>|
| `oci.from` | Location of OCI image.|
| `oci.pull-policy` | Pull policy. Default is `newer`.<br><br>See `--pull` in `buildah-from(1)` for details.<br>|
| `oci.tag` | OCI image tag.|
| `<appspec>.labels.<label>` | The value of the label.|
| `<appspec>.network.mode` | Same as `network.mode`, but by application. It takes precedence when defined.|
| `<appspec>.oci.from` | Same as `oci.from`, but by application. It takes precedence when defined.|
| `<appspec>.oci.pull-policy` | Same as `oci.pull-policy`, but by application. It takes precedence when defined.|
| `<appspec>.oci.tag` | Same as `oci.tag`, but by application. It takes precedence when defined.|
| `users.<uid>.perms` | A space-separated list of "permissions" granted to a specific user.<br><br>The implemented "permissions" are presented below:<br><br>* `enable_3d`<br>* `webcam`<br>* `usb`<br>* `sound`<br><br>For a description of any of them, consult `${X11APPJAIL_APPNAME}:${X11APPJAIL_PROFILE}.allow.<permission>` in the "[User Attributes](#user-attributes)" section.<br>|
| `network.security-group.tables` | If `network.mode` is set to `virtualnet`, this attribute specifies a space-separated list of `pf(4)` tables to which the jail will be added using Security Group hooks.<br><br>If you are going to add additional labels related to Security Groups, do not include `security-group:1`, as this attribute already include it.<br><br>See also: https://github.com/DtxdF/AppJail/wiki/filter<br>|
| `mount.system-fonts` | Read-only mounts the fonts system inside the jail, configure Fontconfig, and rebuild the font cache.|
| `network.virtualnet` | Specify the virtual network to be used when `network.mode` is set to `virtualnet`. If not specified, no virtual network is defined, so the default one is used.|
| `widevine.accept.license` | Install Widevine CDM plugin.|

## OCI Configuration

```yaml
build:
  variants:
    - tag: 15.1
      containerfile: Containerfile
      aliases: ["latest"]
      default: true
      args:
        FREEBSD_RELEASE: "15.1"
        NO_PKGCLEAN: "1"
      cache_dirs: ["pkgcache0:/var/cache/pkg"]
```
