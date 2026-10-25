# Minecraft Server on Proxmox

A self-hosted Minecraft server running in a [LXC container / VM] on Proxmox VE. This repo documents the setup, configuration, and maintenance so it can be reproduced from scratch.

![Architecture diagram](images/architecture.png)

## Overview

| Item | Details |
|------|---------|
| Hypervisor | Proxmox VE [version] |
| Guest type | [LXC container / VM] |
| Guest OS | [Debian 12 / Ubuntu 24.04] |
| Server software | [Paper / Vanilla / Fabric] [Minecraft version] |
| Java version | [Java 21] |
| Resources | [2 vCPU, 4 GB RAM, 20 GB disk] |
| Network | [Static IP 192.168.x.x on vmbr0] |
| Access | [LAN only / Port forward / VPN / Tunnel] |

## Why I Built This

[Two or three sentences on your motivation. For example: wanted to learn virtualization, practice Linux administration, and host a server for friends while keeping everything isolated and backed up.]

## Architecture

```
Internet / LAN
      |
  [Router / Firewall]
      |
  Proxmox Host (vmbr0)
      |
  Minecraft Container
      |-- Java process (managed by systemd)
      |-- Daily backups to [local storage / NAS]
```

[Briefly explain the decisions here, such as why LXC over a VM, why Paper over Vanilla, and how resources were sized.]

## Repository Structure

```
minecraft-proxmox/
├── README.md
├── docs/
│   ├── setup.md              # Full build walkthrough
│   ├── backups.md            # Backup and restore procedures
│   └── troubleshooting.md    # Common issues and fixes
├── config/
│   ├── server.properties.example
│   └── minecraft.service     # systemd unit file
├── scripts/
│   ├── start.sh
│   └── backup.sh
├── images/
│   └── architecture.png
└── .gitignore
```

## Quick Start

### 1. Create the container on Proxmox

1. Download a template: `pveam update && pveam download local [template-name]`
2. Create the container with [X] cores, [X] GB RAM, and [X] GB storage.
3. Assign a static IP and start the container.

Full details are in [docs/setup.md](docs/setup.md).

### 2. Install Java and prepare the server

```bash
apt update && apt install -y openjdk-21-jre-headless
adduser --system --group --home /opt/minecraft minecraft
```

### 3. Download the server

Download [server software] version [x.x.x] from [official source link] and place the jar in `/opt/minecraft/`. The jar is not included in this repo.

### 4. Configure

```bash
cp config/server.properties.example /opt/minecraft/server.properties
```

Edit the file for your needs, and accept the EULA by setting `eula=true` in `eula.txt`.

### 5. Run as a service

```bash
cp config/minecraft.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable --now minecraft
```

## Management

| Task | Command |
|------|---------|
| Start | `systemctl start minecraft` |
| Stop | `systemctl stop minecraft` |
| View logs | `journalctl -u minecraft -f` |
| Run a backup | `./scripts/backup.sh` |

## Backups

- **Method:** [Proxmox vzdump snapshots / script-based tar archives / both]
- **Schedule:** [Daily at 3:00 AM]
- **Retention:** [7 daily, 4 weekly]
- **Destination:** [Local storage / NAS / offsite]

See [docs/backups.md](docs/backups.md) for restore steps.

## Security Notes

- [Server is only exposed via ___ (port forward / VPN / Cloudflare Tunnel)]
- [Firewall rules limit access to port 25565]
- [Whitelist enabled, RCON disabled or bound to localhost]
- [Container runs as an unprivileged user]
- Secrets, player data, and IP addresses are excluded from this repo.

## What I Learned

- [Example: Proxmox networking and bridge configuration]
- [Example: Managing long-running services with systemd]
- [Example: Designing a backup and restore strategy]
- [Example: JVM memory tuning for game servers]

## Future Improvements

- [ ] [Monitoring with Grafana / Prometheus]
- [ ] [Automated updates]
- [ ] [Offsite backups]
- [ ] [Discord notifications for server status]

## License

[MIT](LICENSE)
