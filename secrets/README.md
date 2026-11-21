# Local-only Docker secrets

**Do not commit real credentials here.** This public repository no longer tracks the `*.secret` files that previously lived in this folder.

The live Portainer `docker-compose.yml` expects PIA credentials to exist as **host files** at:

- `${SECRETS_DIR}/openvpn_user.secret`
- `${SECRETS_DIR}/openvpn_password.secret`

For example, set `SECRETS_DIR=/opt/arr-stack/secrets` in Portainer and create those files on Ubuntu with appropriately restricted permissions. Keep the real contents off GitHub.

The alternative Compose examples may reference additional local secret files. See [SECURITY.md](../SECURITY.md).

**If values in older Git commits were real, rotate them.** Deleting files in a new commit does not remove them from previous public Git history.
