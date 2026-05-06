# Pad on Unraid

Run [Pad](https://getpad.dev) on your Unraid server — local-first project management for developers and AI coding agents. Single Go binary, SQLite by default, your data stays on your box.

The complete install walkthrough lives at <https://getpad.dev/docs/self-hosting/unraid>. This README is for **maintainers and operators** who want notes specific to running Pad on Unraid.

## Prerequisites

- Unraid 6.12 or later
- Community Applications plugin (for Route A install)

## Form fields

| Field | Default | What it does |
|---|---|---|
| **WebUI Port** | `7777` | Host port Pad listens on. Change if 7777 is in use. |
| **Appdata** | `/mnt/user/appdata/pad/` | Where Pad stores its database, attachments, encryption key, logs, and config. |
| **PUID** | `99` | UID the pad process runs as inside the container. `99` = Unraid `nobody`. |
| **PGID** | `100` | GID similarly. `100` = Unraid `users`. |
| **Log Level** | `info` | Verbosity for the server log. Set to `debug` when troubleshooting. |
| **Bypass Setup Token** | `false` | If `true`, the first admin can be created directly via `http://<your-tower>:7777/setup` — no bootstrap token required. Only safe on trusted networks (LAN, Tailscale). See first-run setup notes below. |
| **Public URL** | (empty) | Set if reverse-proxying. Required so emailed invitation links point at the real hostname. |
| **Maileroo API Key** | (empty) | Optional. Enables email invitations. Without it, invites use copyable join codes. |
| **Email From** | (empty) | Sender address (required if Maileroo key is set). |
| **Email From Name** | `Pad` | Display name on outbound emails. |

PUID/PGID are validated by the entrypoint — `0` (root), empty, and non-numeric values are rejected with a clear error. The default `99:100` matches Unraid's `nobody:users` convention; you should rarely need to change them.

## First-run setup

Two paths to the first admin account:

- **Token from logs (default):** the container generates a one-time bootstrap token at first start and prints it inside a banner you grab from the **Logs** button. Look for `Pad first-run setup`. Copy the URL (looks like `http://<your-tower>:7777/setup#token=<TOKEN>`), paste in your browser, fill in email/name/password, you're in. The `#token=` portion is a URL fragment — fragments are browser-only, never transmitted to a server, so the token can't end up in proxy logs or browser history shared by URL-shortener tools.

- **Open setup mode:** flip the **Bypass Setup Token** field to `true` (env var `PAD_BYPASS_SETUP_TOKEN=true`). Browse to `http://<your-tower>:7777/setup` and create the first admin directly — no token required. Convenient on trusted networks (LAN-only / Tailscale-only / firewalled). The flag has no effect once the first admin exists.

WARNING: leaving **Bypass Setup Token** on while the WebUI port is exposed to the open internet means anyone who reaches it first can claim the admin account. Use it on networks you trust.

## Where your data lives

All persistent state lives under your appdata path (default `/mnt/user/appdata/pad/`):

| Path | What | Critical? |
| --- | --- | --- |
| `pad.db` + `pad.db-wal` + `pad.db-shm` | SQLite database | yes |
| `encryption.key` | Encryption key for sensitive fields (TOTP seeds, OAuth tokens) | **yes — losing this bricks encrypted data** |
| `attachments/` | Uploaded attachment blobs | yes |
| `logs/server.log` | Server log | nice-to-have |
| `config.toml` | Workspace config | nice-to-have |
| `pad.pid` | PID file | ephemeral |
| `.bootstrap-token` | First-run setup token (auto-deleted on first admin claim) | one-time |

One mount, complete coverage.

## Backups

Stop the container first so SQLite isn't mid-write. Then:

```bash
# Backup
tar -C /mnt/user/appdata -czf "pad-backup-$(date +%F).tar.gz" pad/

# Restore
tar -C /mnt/user/appdata -xzf "pad-backup-2026-05-06.tar.gz"
```

The `-C` flag makes both archive and restore relative to `/mnt/user/appdata`, so the `pad/` directory inside the tarball always lands at `/mnt/user/appdata/pad/` regardless of where you run the command. The container's entrypoint runs a `chown -R` to your configured `PUID:PGID` on every start, so PUID/PGID don't have to match between source and destination Unraid hosts.

## Reverse proxy

If you front Pad with **SWAG** or **NGINX Proxy Manager** (recommended for HTTPS + a real hostname):

1. In the template's **Public URL** field, enter your external URL: `https://pad.example.com`.
2. In your reverse proxy, point `pad.example.com` at `<unraid-host>:7777`.
3. Standard reverse-proxy headers are fine — Pad doesn't need any special config.

The **Public URL** is required if you want emailed invitations to point at your real hostname.

## Email (optional)

If you want Pad to send workspace invitations by email, fill in:

- **Maileroo API Key** — sign up at <https://maileroo.com> (free tier is fine).
- **Email From** — sender address on a domain you control.
- **Email From Name** — display name (defaults to "Pad").

Without these, invitations fall back to copyable join codes you paste into the invitee's CLI. Not worse — just different.

## Source

Runtime image: [`ghcr.io/perpetualsoftware/pad:latest`](https://github.com/PerpetualSoftware/pad/pkgs/container/pad)
Pad source: <https://github.com/PerpetualSoftware/pad>
Bug reports: <https://github.com/PerpetualSoftware/pad/issues>
Full docs: <https://getpad.dev/docs/self-hosting/unraid>
