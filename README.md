# Rathole Tunnel Expanded

A simple interactive bash script to install and manage **rathole** tunnels (IRAN/Kharej), generate `.toml` configs, create `systemd` services, and optionally enable a **log-based healthcheck** to auto-restart stuck tunnels.

> Works best on Ubuntu/Debian-based servers.

---

## Features

- Install rathole core (download + extract)
- Create IRAN (server) tunnel configs
- Create KHAREJ (client) tunnel configs
- Generate and manage `systemd` services automatically
- Optional **log-based healthcheck** (detects common stuck patterns and restarts the service)
- Optional keepalive tuning (`keepalive_secs`, `keepalive_interval`) during tunnel creation
- Basic tunnel management menu (start/stop/restart/status/logs)

---

## Quick Install

```bash
bash <(curl -Ls --ipv4 https://raw.githubusercontent.com/ach1992/rathole-tunnel-expanded/main/rathole.sh)
```

> If you renamed the repo or file, update the URL accordingly.

---

## Usage

1. Run the script
2. Install core (if not installed)
3. Configure IRAN server (server mode)
4. Configure KHAREJ server (client mode)
5. Use the tunnel management menu to start/stop/restart/view status/logs

---

## Healthcheck (Log-based)

The healthcheck is designed to restart the tunnel even when the service is still running but the tunnel is **stuck**.

It scans recent logs for patterns like:

- `Failed to run the data channel`
- `Heartbeat timed out`
- `Connection timed out`

You can enable it:

- Automatically after creating a KHAREJ tunnel (prompt)
- From the tunnel management menu (optional)

---

## Paths

### systemd services

- `/etc/systemd/system/rathole-iran*.service`
- `/etc/systemd/system/rathole-kharej*.service`

### Config files

- `/root/rathole-core/iran*.toml`
- `/root/rathole-core/kharej*.toml`

---

## Disclaimer

This project modifies system configuration (systemd services, optional cron/healthcheck).  
Use at your own risk and review the script before running it on production servers.

---

## License

MIT (or your preferred license)

---

⭐ If this project helped you, please **star** the repository!
