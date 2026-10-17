<div align="center">

<img src="logo/norypt-icon-512_No_background.png" alt="NORYPT" width="180" />

# NORYPT Privacy for Puli AX

**Network-identifier privacy for the GL.iNet Puli AX (GL-XE3000) travel router**

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform: OpenWrt](https://img.shields.io/badge/platform-OpenWrt%20%2F%20GL.iNet-1f6feb.svg)](#supported-hardware)
[![Hardware: GL-XE3000](https://img.shields.io/badge/hardware-GL--XE3000-3fb950.svg)](#supported-hardware)

</div>

---

> [!IMPORTANT]
> **Read the [Legal Notice](#legal-notice) before installing.** Changing a cellular modem's IMEI
> is a criminal offence in some countries. You are responsible for checking and following the
> laws of the country you live in and every country you take the router to. If it is not clearly
> legal where you are, keep IMEI changes switched off (`norypt config set randomize_imei off`).

NORYPT Privacy is an open-source OpenWrt add-on for the GL.iNet Puli AX (GL-XE3000), a router
you own. It cuts down on long-term passive tracking of the router by giving it fresh network
identifiers. Public Wi-Fi operators, hotel and venue networks and other third parties often log
these identifiers to recognise the same device from one session to the next.

On each boot, or when you ask it to, NORYPT can:

- give the router's own Wi-Fi access points new **BSSIDs** (Wi-Fi MAC addresses)
- give the WAN port a new **MAC address**
- give the router's own built-in modem a new **IMEI**, only where that is lawful (see the [Legal Notice](#legal-notice))
- clear the router's **volatile logs and DHCP leases**

It installs as a normal OpenWrt service with a local web panel, a command-line tool and an
offline installer. Nothing runs in the cloud and it makes no outbound connections.

This is the same idea as the MAC-address randomization that iOS, Android, Windows and macOS
already do on Wi-Fi, applied to a travel router.

---

## Table of Contents

1. [Legal Notice](#legal-notice)
2. [Acceptable Use](#acceptable-use)
3. [Features](#features)
4. [Supported Hardware](#supported-hardware)
5. [Installation — Online](#installation--online)
6. [Installation — Offline](#installation--offline)
7. [Web Panel](#web-panel)
8. [CLI Reference](#cli-reference)
9. [Configuration](#configuration)
10. [How It Works](#how-it-works)
11. [Project Structure](#project-structure)
12. [Security Notes](#security-notes)
13. [Uninstall](#uninstall)
14. [Third-Party Software](#third-party-software)
15. [Disclaimer and Trademarks](#disclaimer-and-trademarks)
16. [License](#license)

---

## Legal Notice

**IMEI changes.** Many jurisdictions regulate changing a mobile device's IMEI, and some make it
a crime. Examples include the United Kingdom (Mobile Telephones (Re-programming) Act 2002) and
India (Telecommunications Act 2023). Other countries have similar rules, and carriers' terms of
service often ban it as well. This feature is provided for **research, testing and personal
privacy only, and only where the law allows it**. If you are not sure, switch it off:

```sh
norypt config set randomize_imei off
```

**MAC address / BSSID changes.** Changing your own device's MAC address is legal in most places
and is built into all modern operating systems. Some networks' terms of use may still forbid it.

**Your responsibility.** By installing or using this software you confirm that:

- you own the router, or have the owner's permission, to change it;
- you will follow all laws and regulations that apply to you, and your mobile carrier's terms;
- you will not use it for any of the purposes listed under [Acceptable Use](#acceptable-use).

The authors give no legal advice and accept no liability for misuse. If in doubt, talk to a
qualified lawyer in your jurisdiction.

---

## Acceptable Use

NORYPT is a **privacy** tool for hardware **you own**. It must **not** be used to:

- impersonate or clone another person's device or identifier;
- get around a block or blacklist on a lost, stolen or barred device, or any network ban;
- commit fraud, avoid paying for service, or break a carrier's or network's terms;
- avoid or get in the way of lawful investigations, or any other unlawful activity;
- touch any device or network you are not authorized to change.

The project does **not** include, and will not accept, anything that does these things. Issues
or pull requests asking for them will be closed.

---

## Features

| Capability | Description |
|---|---|
| **Wi-Fi BSSID randomization** | Gives the router's 2.4 GHz and 5 GHz access points new unicast MAC addresses, using common chipset vendor prefixes (OUIs). Applied with `ip link` and saved to UCI. |
| **WAN MAC randomization** | Sets a new MAC on the WAN port (`eth0`) before DHCP runs. A hotplug hook puts it back whenever `netifd` brings WAN up again. |
| **Modem IMEI change (opt-out)** | Builds a correctly formatted, Luhn-valid IMEI for the router's built-in modem. You can choose the device brand and region. **Only use it where it is lawful.** You can switch it off in the panel or the CLI. |
| **Log clearing** | Clears the router's own volatile logs (`/var/log`, `/tmp/log`), DHCP leases and UCI temp files, then restarts the syslog service. |
| **Runs at boot** | Runs at `START=19`, before `network` (`START=20`), so new identifiers are in place before the router connects to anything. |
| **Local web panel** | `http://192.168.8.1/norypt/` shows status and lets you toggle features, randomize by hand and see recent events. LAN-only and CSRF-protected. |
| **CLI tool** | `norypt status / randomize / wipe-logs / config / service / version`. |
| **Survives firmware upgrades** | Adds its files to `/etc/sysupgrade.conf` so a firmware upgrade keeps them. |
| **Online and offline install** | One-command install over SSH, or a fully offline install with the bundled `.ipk` packages. |
| **Web server auto-detection** | Works with `uhttpd` (vanilla OpenWrt) and `nginx + fcgiwrap` (GL.iNet firmware 4.x). |

---

## Supported Hardware

| Device / firmware | Status |
|---|---|
| GL.iNet Puli AX (GL-XE3000) | Primary target |
| GL.iNet firmware 4.5+ (`nginx`) | Supported |
| GL.iNet firmware 4.x (< 4.5) | Supported |
| Vanilla OpenWrt (MediaTek Filogic 820) | Supported |

The built-in cellular modem is reached through `ubus` (GL.iNet firmware) or its AT serial port
(vanilla OpenWrt).

---

## Installation — Online

Use this method when the router has internet access.

1. Connect your computer to the router (Wi-Fi or LAN cable).
2. Open an SSH session to the router (the default GL.iNet LAN address is `192.168.8.1`; the
   password is the admin password you set in the GL.iNet web UI):

   ```sh
   ssh root@192.168.8.1
   ```

3. Run the installer on the router:

   ```sh
   wget -O - https://raw.githubusercontent.com/norypt-website/Puli-AX-GL-XE3000/main/install.sh | sh
   ```

   If you'd like to read the script before running it:

   ```sh
   wget -O /tmp/install.sh https://raw.githubusercontent.com/norypt-website/Puli-AX-GL-XE3000/main/install.sh
   less /tmp/install.sh
   sh /tmp/install.sh
   ```

4. Open **http://192.168.8.1/norypt/** in your browser.

The installer:

1. Detects the firmware (GL.iNet or vanilla OpenWrt).
2. Installs any missing dependencies with `opkg` (`uqmi`, `bash`, `coreutils-shuf`, plus
   `fcgiwrap` on nginx firmware).
3. Downloads the modules, databases, web panel, CGI, init script, hotplug hook and CLI from
   this repository.
4. Enables and starts the `norypt` service.
5. Sets up the panel route (`uhttpd` redirect or `nginx` location block).
6. Adds the NORYPT files to `/etc/sysupgrade.conf`.

---

## Installation — Offline

Use this method when the router has **no internet access**. Every dependency it needs ships in
[`offline/deps/`](offline/deps), built for OpenWrt 23.05 / `aarch64_cortex-a53`.

**1. Get the project onto your computer** (do this while you have internet):

- with git:

  ```sh
  git clone https://github.com/norypt-website/Puli-AX-GL-XE3000.git
  ```

- or without git: on the repository page click **Code → Download ZIP**, then unzip it. Rename
  the unzipped folder `Puli-AX-GL-XE3000-main` to `Puli-AX-GL-XE3000`.

**2. Copy the whole folder to the router.** Copy the full folder, not just `offline/`:

```sh
scp -O -r Puli-AX-GL-XE3000 root@192.168.8.1:/tmp/
```

> `-O` makes scp use the classic protocol. GL.iNet firmware uses Dropbear SSH, which doesn't
> support the SFTP mode that newer OpenSSH (Windows 10/11, macOS, Linux) uses by default.
> On Windows you can also use WinSCP with the **SCP** protocol.

**3. Run the offline installer on the router:**

```sh
ssh root@192.168.8.1
sh /tmp/Puli-AX-GL-XE3000/offline/install-offline.sh
```

**4. Open http://192.168.8.1/norypt/** in your browser.

`/tmp` is RAM, so the copied folder disappears on reboot. The installed files live in
`/etc/norypt` and stay.

**Bundled packages** (OpenWrt 23.05, `aarch64_cortex-a53`):

| Package | Version | License |
|---|---|---|
| `bash` | 5.2.15-1 | GPL-3.0-or-later |
| `coreutils` | 9.3-1 | GPL-3.0-or-later |
| `coreutils-shuf` | 9.3-1 | GPL-3.0-or-later |
| `libreadline8` | 8.2-1 | GPL-3.0-or-later |
| `libncurses6` | 6.4-2 | MIT |
| `libubox20230523` | 2023-05-23 | ISC |
| `libblobmsg-json20230523` | 2023-05-23 | ISC |
| `uqmi` | 2022-10-20 | GPL-2.0 |
| `wwan` | 2019-04-29 | GPL-2.0 |

> If your firmware is a different build, the bundled `.ipk` files may clash with what's
> installed. Use the online method, or download matching packages from
> <https://downloads.openwrt.org/releases/>.

### Build as an OpenWrt package (optional)

```sh
git clone https://github.com/norypt-website/Puli-AX-GL-XE3000.git /path/to/openwrt/package/norypt-privacy
cd /path/to/openwrt
make package/norypt-privacy/compile V=s
# then copy the resulting norypt-privacy_1.0.0-1_all.ipk to the router and:
opkg install /tmp/norypt-privacy_1.0.0-1_all.ipk
```

---

## Web Panel

Open `http://192.168.8.1/norypt/` from a device on the router's LAN.

- **Current Identities**: BSSID 2.4 GHz, BSSID 5 GHz, WAN MAC, modem IMEI and cellular status.
- **Feature Toggles**: switch each randomization, log clearing and run-on-boot on or off.
- **IMEI Selection**: brand and region of the generated identifier.
- **Manual Controls**: randomize one identifier or all of them, or clear logs.
- **Recent Events**: the last 10 boot and manual randomization events.

---

## CLI Reference

```
norypt status                          Show identifiers, cellular status, config
norypt randomize [imei|bssid|wan|all]  Randomize identifiers (default: all enabled)
norypt wipe-logs                       Clear the router's volatile logs
norypt config show                     Show current UCI config
norypt config set <key> <value>        Change a setting
norypt service <start|stop|restart|status>
norypt version                         Show version and firmware info
norypt help                            Show help
```

Examples:

```sh
norypt status
norypt randomize bssid
norypt config set randomize_imei off
```

---

## Configuration

Settings are stored in UCI at `/etc/config/norypt`:

| Key | Default | Description |
|---|---|---|
| `enabled` | `1` | Master on/off switch |
| `randomize_imei` | `1` | Change the modem IMEI on boot. **Set to `0` unless it is lawful where you are.** |
| `randomize_bssid` | `1` | Randomize Wi-Fi BSSIDs on boot |
| `randomize_wan` | `1` | Randomize WAN MAC on boot |
| `wipe_logs` | `1` | Clear logs on boot |
| `wipe_dhcp` | `1` | Clear DHCP leases on boot |
| `on_boot` | `1` | Run automatically at boot |
| `settle_delay` | `3` | Seconds to wait after the modem starts |
| `cellular_timeout` | `60` | Seconds to wait for a cellular IP |
| `log_history` | `10` | Number of events kept in the panel history |
| `imei_brand` | `all` | Brand used for the IMEI: `all`, `apple`, `samsung`, `xiaomi`, `motorola` |
| `imei_region` | `auto` | Region: `auto` (from the SIM), `na`, `eu`, `asia`, `latam`, `global` |
| `imei_default_region` | `eu` | Region to use when `auto` can't work it out |

---

## How It Works

1. **BSSID**: picks a vendor prefix from `oui-wifi.db` (48 entries) and adds 3 random bytes,
   with the multicast and locally-administered bits cleared. It is applied with `ip link`,
   saved to UCI, and the Wi-Fi is restarted so `hostapd` uses the new address.
2. **WAN MAC**: the same method with `oui-wan.db` (45 entries), applied to `eth0` and saved to
   `network.wan.macaddr`. The `99-norypt-wan` hotplug hook applies it again on every `ifup wan`.
3. **IMEI** (only if enabled): picks a type-allocation code (TAC) from `tac.db` (92 entries,
   filtered by your brand and region settings), adds a random 6-digit serial and calculates the
   Luhn check digit. It writes the IMEI to the router's own modem with standard AT commands,
   then reads it back to check it. A generated number could match one already in use, which is
   one more reason to use this only where it is lawful.
4. **Log clearing**: deletes files in `/var/log` and `/tmp/log`, the DHCP leases and the
   `/tmp/.uci` deltas, then restarts the log service to empty the in-memory buffer.
5. **Boot order**: the procd init script runs at `START=19`, before `network`.

All randomness comes from the kernel (`/proc/sys/kernel/random/uuid`). Everything runs on the
router; nothing is sent anywhere.

---

## Project Structure

```
Puli-AX-GL-XE3000/
├── src/
│   ├── modules/        # boot orchestrator and one script per feature
│   ├── db/             # OUI and TAC prefix databases
│   ├── bin/norypt      # CLI (/usr/bin/norypt)
│   ├── cgi-bin/        # web panel backend (CSRF-protected)
│   ├── www/            # web panel frontend
│   ├── init.d/norypt   # procd service (START=19)
│   ├── hotplug.d/      # WAN MAC hook
│   ├── uci-defaults/   # first-boot / sysupgrade hook
│   └── config/norypt   # default UCI config
├── offline/
│   ├── install-offline.sh
│   └── deps/           # bundled .ipk dependencies + notices
├── logo/
├── install.sh          # online installer
├── uninstall.sh
├── Makefile            # OpenWrt SDK package definition
└── LICENSE
```

---

## Security Notes

- The panel is **LAN-only** and is not exposed on WAN or cellular.
- Every write action needs a per-session CSRF token (`X-Norypt-Token` header), which is
  checked on the server.
- Config changes go through a server-side allowlist of UCI keys.
- It makes **no outbound connections** at runtime.
- Found a security issue? Please open a GitHub issue that leaves out exploit details, and we
  will arrange a private channel.

---

## Uninstall

Online:

```sh
wget -O - https://raw.githubusercontent.com/norypt-website/Puli-AX-GL-XE3000/main/uninstall.sh | sh
```

Offline (with the folder copied to the router as above):

```sh
sh /tmp/Puli-AX-GL-XE3000/uninstall.sh
```

The uninstaller stops the service and removes all NORYPT files, the web server config and the
sysupgrade entries. Identifiers that were already changed keep their current values until the
next reboot, or until you reset them in the GL.iNet UI (Wi-Fi and WAN MAC).

---

## Third-Party Software

The `.ipk` files in [`offline/deps/`](offline/deps) are unmodified binary packages from the
OpenWrt project, redistributed under their own licenses (listed above). The source code for
each one is available from OpenWrt. See
[`offline/deps/THIRD_PARTY_NOTICES.md`](offline/deps/THIRD_PARTY_NOTICES.md).

---

## Disclaimer and Trademarks

This software is provided **"as is", without warranty of any kind**. Changing router and modem
settings can cause loss of connectivity, may void your warranty, and in rare cases may need a
firmware reset to recover. Use it at your own risk.

NORYPT is an independent project. It is **not affiliated with, endorsed by or sponsored by**
GL.iNet, Quectel, MediaTek, the OpenWrt project or any mobile carrier or device manufacturer.
All product names and trademarks belong to their respective owners and are used here only to
describe compatibility.

---

## License

[MIT](LICENSE). Bundled third-party packages keep their own licenses (see
[Third-Party Software](#third-party-software)).

Issues and pull requests are welcome; please keep them within the [Acceptable Use](#acceptable-use) policy.
