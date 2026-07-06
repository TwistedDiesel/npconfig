# npconfig

`npconfig` is a utility for configuring **netplan** through an **ifconfig-compatible CLI** and an interactive **curses menu**. It is designed to be more user-friendly than fighting YAML on a lone server with no network.

No sysadmin should have to bring a netplan manual just to bring networking up.

## Features

- **ifconfig-style commands** for quick provisioning
- **Wi-Fi setup** (scan, connect, hidden SSID, DHCP/static)
- **Interactive menu** (`npconfig menu`) using `dialog` or `whiptail`
- Writes persistent config to `/etc/netplan/99-npconfig-IFACE.yaml`
- Uses **NetworkManager** renderer (standard on Ubuntu desktop/server with NM)

## Requirements

- Linux with **netplan** and **iproute2** (`ip`)
- **root** / `sudo`
- For Wi-Fi scan/menu: **NetworkManager** (`nmcli`) recommended, or **iw** as fallback
- For interactive menu: **whiptail** or **dialog**

```bash
sudo apt-get install -y netplan.io iproute2 network-manager whiptail
```

`ifconfig` is **not** required.

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/TwistedDiesel/npconfig/main/npconfig -o npconfig
chmod +x npconfig
sudo install -m 755 npconfig /usr/local/bin/npconfig
```

Or clone and install:

```bash
git clone https://github.com/TwistedDiesel/npconfig.git
cd npconfig
sudo install -m 755 npconfig /usr/local/bin/npconfig
```

## Interactive menu (recommended for new hosts)

```bash
sudo npconfig menu
```

Menu options:

- Configure Ethernet (DHCP / static / up / down / status)
- Configure Wi-Fi (scan & connect, hidden SSID, DHCP/static)
- Show interface status
- Apply netplan

Aliases: `npconfig --menu`, `npconfig -m`

## CLI usage (ifconfig-compatible)

```bash
# Show interfaces
npconfig
npconfig -a
npconfig eth0

# Link state
npconfig eth0 up
npconfig eth0 down

# DHCP (persisted via netplan)
npconfig eth0 inet dhcp

# Static IPv4
npconfig eth0 192.168.1.50 netmask 255.255.255.0 up
```

## Wi-Fi CLI

```bash
# Scan
npconfig wlan0 wifi scan

# Open network + DHCP
npconfig wlan0 wifi GuestNetwork open inet dhcp up

# WPA network + DHCP
npconfig wlan0 wifi MySSID passphrase mypassword inet dhcp up

# Static IP on Wi-Fi
npconfig wlan0 wifi MySSID psk mypassword 192.168.1.50 netmask 255.255.255.0 up

# Hidden SSID
npconfig wlan0 wifi --hidden SecretSSID passphrase mypassword inet dhcp up
```

## How it works

`npconfig` writes per-interface files:

```text
/etc/netplan/99-npconfig-<iface>.yaml
```

Then runs:

```bash
netplan generate
netplan apply
```

These files are intentionally prefixed with `99-` so they override simpler defaults while remaining easy to find and remove.

## Example netplan output (Wi-Fi DHCP)

```yaml
network:
  version: 2
  renderer: NetworkManager
  wifis:
    wlan0:
      dhcp4: true
      access-points:
        "MySSID":
          password: "mypassword"
```

## License

GPL-3.0 — see [LICENSE](LICENSE).
