# Rocki

Firmware and information about the [Rocki Play](https://www.kickstarter.com/projects/rocki/rocki-wifi-music-system-from-every-phone-to-all-sp) device.

## Files in this repository

`dlna_upnp.tar.gz`: The latest "firmware" archive from the Wayback Machine.

`root.img`: An image of the cramfs ROM filesystem (`/dev/root`).

## Rocki Play OS

### Filesystems

The Rocki Play has a read-only cramfs root filesystem (`/dev/root`), and a read-write flash storage filesystem (`/dev/mtdblock2`).

All the audio and web-related programs are stored in flash storage, as well as user data.

#### Flash storage

```
dev: size erasesize name
mtd0: 00400000 00020000 "logo"
mtd1: 02800000 00020000 "root"
mtd2: 04a00000 00020000 "yaffs"
```

### Boot process

Upon boot, the root filesystem is mounted, and the scripts in `/etc/init.d` are ran. The main init script, `/etc/init.d/rcS`, mounts the flash storage to `/mnt/sd` and launches an initialization script (`/mnt/sd/init`). This flash storage init script is in charge of displaying the LED boot animation, as well as starting all the audio and web-related services.

### Updating

When a "firmware" tarball is uploaded via the CGI upload mechanism, `/mnt/sd/update_sh` backs up user data, copies the new programs and configuration files from the tarball into the flash storage, and restores user data.

### Environment

#### Kernel

`Linux version 2.6.32.27 (root@localhost.localdomain) (gcc version 4.3.2 (Sourcery G++ Lite 2008q3-72) ) #210 Thu Apr 3 13:35:42 CST 2014`

#### Busybox

`BusyBox v1.18.4 (2011-04-13 15:26:32 CST) multi-call binary.`

#### Mounts

```
rootfs on / type rootfs (rw)
/dev/root on / type cramfs (ro,relatime)
proc on /proc type proc (rw,relatime)
sysfs on /sys type sysfs (rw,relatime)
tmpfs on /mnt/sda type tmpfs (rw,relatime)
mdev on /dev type tmpfs (rw,relatime)
/dev/mtdblock2 on /mnt/sd type yaffs2 (rw,relatime)
tmpfs on /tmp type tmpfs (rw,relatime)
tmfps on /etc type tmpfs (rw,relatime)
tmfps on /wpa_net/run/wpa_supplicant type tmpfs (rw,relatime)
tmpfs on /home type tmpfs (rw,relatime)
```

#### Networking

##### Hostname

The hostname is `ASAP1826T`, set by `/etc/init.d/rcS` on boot.

##### WPA

`wpa_supplicant v0.7.3`

##### Interfaces

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether a0:b1:c2:d3:e4:f5 brd ff:ff:ff:ff:ff:ff
```

#### Open ports

`netstat -tulpn | grep LISTEN`

| Port  | Protocol      | Process / version                       | Comment                                                      |
| ----- | ------------- | --------------------------------------- | ------------------------------------------------------------ |
| 80    | HTTP          | `boa` / `0.94.13`                       | The main configuration site.                                 |
| 1234  | HTTP/JSON-RPC | `wifi_key_det`                          | `/mnt/sd/dlna_upnp/networkconfig/nettools/wifi_key_det`      |
| 1337  | Shell         | `ushare` / `(1.1a) (Built May 22 2013)` | The [uShare](https://ushare.geexbox.org) telnet console.     |
| 5000  | HTTP          | `spotify-rocki`                         | Spotify Connect.                                             |
| 5002  | RTSP/AirPlay  | `airaudio`                              | AirPlay.                                                     |
| 8888  | HTTP          | `onairconnectarm`                       | `/mnt/sd/dlna_upnp/onairconnectarm`                          |
| 49152 | HTTP          | `ushare`                                | UPnP/DLNA, and the uShare configuration site (at `/web/ushare.html`). |
| 49154 | DLNA          | `gmediarenderer`                        | DLNA.                                                        |

## Shell access

A root shell can be accessed at `/shell.html`. This is a simple client that executes commands by performing a HTTP GET request to the following URL: `/cgi-bin/rocki.cgi?ShellCMD=<command>`.

### Pulling disk images

While FTP, TFTP, and telnet clients and servers are included in the rootfs, I could not get any of these to work correctly.

The shell API can be used like so, replacing `<block_name>` and `<output_file>` with the block file name and output file name:

```
curl 'http://192.168.1.64/cgi-bin/rocki.cgi?ShellCMD=base64%20%2Fdev%2F<block_name>' | base64 -do <output_file>
```

`base64` is required as the shell API mangles binary data. Unfortunaly, this is a significant bottleneck.