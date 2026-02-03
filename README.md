# Rathole Tunnel Expanded

A simple interactive bash script to install and manage rathole tunnels (IRAN/Kharej), generate `.toml` configs, create systemd services, and optionally enable a log-based healthcheck to auto-restart stuck tunnels.

---

## Install & Run

One-line install:

```bash
bash <(curl -Ls --ipv4 https://raw.githubusercontent.com/ach1992/rathole-tunnel-expanded/main/rathole.sh)
```

Or clone the repo:

```bash
git clone https://github.com/ach1992/rathole-tunnel-expanded.git
cd rathole-tunnel-expanded
chmod +x rathole.sh
sudo bash rathole.sh
```

> Run as root (or with `sudo`) because the script creates systemd services and writes to system paths.

---

## What it does

- Installs **rathole** (depending on script options/version)
- Creates tunnels with interactive menus:
  - **Iran (Server)**: bind on a port (example: `0.0.0.0:<PORT>`)
  - **Kharej (Client)**: connect to Iran server (example: `IRAN_IP:<PORT>`)
- Generates `.toml` configs automatically
- Creates a **systemd service per tunnel**, for example:
  - `rathole-iran<PORT>.service`
  - `rathole-kharej<PORT>.service`
- Can enable an optional **healthcheck** (systemd timer) per tunnel:
  - Watches logs for repeated tunnel errors
  - Restarts the tunnel service with a cooldown to avoid restart loops

---

## Important paths (script defaults)

Typical files look like this:

- Rathole binary:
  - `/root/rathole-core/rathole`
- Tunnel configs:
  - Iran (server): `/root/rathole-core/iran*.toml`
  - Kharej (client): `/root/rathole-core/kharej*.toml`
- systemd services:
  - `/etc/systemd/system/rathole-iran*.service`
  - `/etc/systemd/system/rathole-kharej*.service`
- Healthcheck script:
  - `/usr/local/bin/rathole-healthcheck.sh`
- Healthcheck systemd template units:
  - `/etc/systemd/system/rathole-healthcheck@.service`
  - `/etc/systemd/system/rathole-healthcheck@.timer`
- Healthcheck cooldown state:
  - `/run/rathole-healthcheck/<service-base>.last_restart_epoch`

> `*` means “any tunnel port/name created by the script”.

---

## Common systemd commands

Replace `<PORT>` with your tunnel port (example: `443`, `8443`, `2053`, ...)

Start / Stop:
```bash
sudo systemctl start rathole-kharej<PORT>.service
sudo systemctl stop rathole-kharej<PORT>.service
```

Restart:
```bash
sudo systemctl restart rathole-kharej<PORT>.service
```

Enable on boot:
```bash
sudo systemctl enable --now rathole-kharej<PORT>.service
```

Disable:
```bash
sudo systemctl disable --now rathole-kharej<PORT>.service
```

Show the final unit file:
```bash
sudo systemctl cat rathole-kharej<PORT>.service
```

---

## Healthcheck (per tunnel)

Healthcheck is enabled **per tunnel** using a systemd timer instance.

For example, for `rathole-kharej<PORT>.service`:

- Timer name: `rathole-healthcheck@rathole-kharej<PORT>.timer`
- Service name: `rathole-healthcheck@rathole-kharej<PORT>.service`

Enable Healthcheck for a tunnel:
```bash
sudo systemctl enable --now rathole-healthcheck@rathole-kharej<PORT>.timer
```

Disable Healthcheck for a tunnel:
```bash
sudo systemctl disable --now rathole-healthcheck@rathole-kharej<PORT>.timer
```

Check timer status:
```bash
sudo systemctl status rathole-healthcheck@rathole-kharej<PORT>.timer --no-pager
```

---

## Logs & Debugging

### 1) Tunnel logs
Last 200 lines:
```bash
sudo journalctl -u rathole-kharej<PORT>.service --no-pager -n 200
```

Follow logs live:
```bash
sudo journalctl -u rathole-kharej<PORT>.service -f
```

### 2) Healthcheck execution logs (per tunnel)
```bash
sudo journalctl -u rathole-healthcheck@rathole-kharej<PORT>.service --no-pager -n 200
```

### 3) Healthcheck “decision” logs (Restart / Cooldown messages)
```bash
sudo journalctl -t rathole-healthcheck --no-pager -n 200
```

### 4) List timers
```bash
sudo systemctl list-timers --all | grep rathole || true
```

---

## Healthcheck settings (optional)

Edit:
```bash
sudo nano /usr/local/bin/rathole-healthcheck.sh
```

Main variables:
- `WINDOW="3 min"` (log search window)
- `THRESHOLD=3` (errors count threshold)
- `COOLDOWN_SECONDS=300` (cooldown between restarts)

After editing, you can test it manually:
```bash
sudo systemctl start rathole-healthcheck@rathole-kharej<PORT>.service
sudo journalctl -t rathole-healthcheck --no-pager -n 50
```

## Disclaimer

This project modifies system configuration (systemd services, optional cron/healthcheck).  
Use at your own risk and review the script before running it on production servers.

---

⭐ If this project helped you, please **star** the repository!
