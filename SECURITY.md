# Docker media server security and recovery guide

This repository's **live Portainer media stack** uses `docker-compose.yml`. Deploy `plex-compose.yml` as a distinct stack after preserving the previous Plex container's logs/configuration. The other Compose files in the repository are alternative examples, **not** part of this live media stack.

## Before merging or deploying the pull request

1. **Preserve evidence and backups.** After the reported Plex vulnerability, save old Plex logs, stack definitions, and `/opt/arr-stack/config/plex`. Never bind that old configuration into the new Plex container. A fresh Plex stack does not prove the Ubuntu host is uncompromised.
2. **Confirm the Ubuntu VM has a reserved LAN IPv4 address of 192.168.1.89.** The media apps and Plex now bind host ports specifically to this address. If it changes, update the Compose files before redeployment. Verify that this IP is already assigned to the host.
3. **Keep UniFi TCP 32400 WAN forwarding disabled** during Plex recovery. Review all other WAN port forwards, UPnP rules, and IPv6 firewall exposure. Binding a published Docker port to a LAN IP does not override a router port-forward.
4. **Verify the existing Portainer environment variables and files** before redeployment: `DATA_DIR`, `SECRETS_DIR`, `PUID`, `PGID`, `TZ`, `PIA_REGION`, `LAN_SUBNET`. Do not paste or commit their actual secret values.
5. **Check stored credentials.** This public repository previously committed files in `secrets/*.secret`. The current branch removes these tracked files and adds a .gitignore rule. That does **not** purge past Git history. If any value was live, revoke/rotate it (PIA, Cloudflare, Sonarr/Radarr API, qBittorrent, and Plex-related credentials) before bringing services online. Consult GitHub's guidance before considering history rewriting.
6. **Back up persistent data separately.** Media config uses host bind mounts such as `/opt/arr-stack/config` and `/opt/arr-stack/downloads`. A Compose edit does not back up or validate those directories.
7. **Review the diff and redeploy intentionally.** Do not automatically pull newer application images during the initial Plex separation. Stop the old `plex` service and preserve its data; ensure the old container name and port are freed before starting the new Plex stack.

## What stays on the Ubuntu host

Set `SECRETS_DIR` in Portainer to a protected directory such as `/opt/arr-stack/secrets`. The live media stack expects the files `openvpn_user.secret` and `openvpn_password.secret` there; these files must be created on the host and must **never** be pushed to this public GitHub repository. Other alternative templates may require additional local secret files. Use restrictive host permissions; don't expose `SECRETS_DIR` to Plex or web dashboards.

The `secrets/*.secret` paths previously tracked in this repository were removed from the branch. `.gitignore` only helps prevent **new accidental commits**; it does not affect any prior copies or revoke tokens.

## Ports and access

The main stack publishes qBittorrent (TCP 8080), Prowlarr (9696), Sonarr (8989), Radarr (7878), and Overseerr (5055) on `192.168.1.89`, rather than `0.0.0.0`. The existing torrent listener TCP/UDP 6881 is also bound to the LAN address. These changes restrict the *host interface*, not access from every device on that LAN, other containers, or WAN traffic forwarded by the router.

Use Tailscale or another authenticated private network for remote administration. Avoid WAN-exposing Portainer, SSH, Sonarr, Radarr, Prowlarr, qBittorrent, File Browser, or Homarr. Use app authentication as an additional layer.

The `FIREWALL_OUTBOUND_SUBNETS=${LAN_SUBNET}` setting on Gluetun creates an exception that allows the VPN network namespace to contact the LAN subnet. This is distinct from VPN kill-switch behavior. Review whether this exception is necessary and narrow it where practical; test qBittorrent Web UI reachability and the VPN kill switch before changing it.

## Homarr Docker socket proxy (separate Portainer stack)

This GitHub repository does **not** manage the Homarr or `docker-socket-proxy` containers. In the proxy's own Portainer deployment, set `POST=0` for a monitoring-only dashboard, keep privileged mode disabled, keep the Docker socket mount read-only, and ensure the proxy port is **not** published to the host or WAN. An `:ro` socket mount does *not* by itself make Docker's API read-only; proxy permissions are essential. Some Homarr actions may stop working when POST is disabled; verify required features.

## New Plex stack

The standalone Plex configuration uses `/opt/arr-stack/config/plex-fresh:/config` (fresh data), `/opt/arr-stack/media:/media:ro` (media read-only) and `/opt/arr-stack/transcode:/transcode`. Ensure writable directories are owned/accessibly writable by UID/GID 1000. It publishes TCP 32400 only on `192.168.1.89`. Keep router forwarding off while investigating the prior incident.

Supply `PLEX_CLAIM` as a temporary Portainer stack environment variable only if required for initial claim; **never commit a real claim token to Git**. Confirm the deployed Plex version contains current security fixes, reset Plex account credentials and sessions, and use individual Plex accounts for friends/family. Decide on remote access only after host recovery and security verification.

## Updates

The Compose files use floating `:latest` image tags. That does not automatically update already-running containers. Establish a routine to back up configuration, pull updates, review releases, apply updates, and verify streaming, downloads, and Gluetun fail-closed behavior. For incident recovery, a clean VM rebuild provides higher assurance than filesystem checks alone.
