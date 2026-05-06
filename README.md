# Perpetual Software · Unraid Templates

[![License](https://img.shields.io/badge/license-Apache%202.0-blue)](LICENSE)

Unraid Community Applications templates for Perpetual Software products. Each app lives in its own subdirectory with the template XML, an icon, and an app-specific README.

## Apps

| App | Directory | Docs | CA Listing |
|-----|-----------|------|------------|
| **Pad** — local-first project management for developers + AI coding agents | [`pad/`](pad/) | [getpad.dev/docs/self-hosting/unraid](https://getpad.dev/docs/self-hosting/unraid) | *pending review* |

## Install

Once a template is approved in the Community Applications app feed, install it via the **Apps** tab in your Unraid web UI — search for the app name. The two routes below are for **manual install** before CA approval lands or for testing template changes.

### Route A — CA Private Folder (recommended)

The closest experience to a CA-approved install. The template appears under CA's **Private** category and uses the same install form as a published app.

```bash
ssh root@<your-tower>
mkdir -p /boot/config/plugins/community.applications/private/perpetualsoftware
wget -O /boot/config/plugins/community.applications/private/perpetualsoftware/pad.xml \
  https://raw.githubusercontent.com/PerpetualSoftware/unraid-templates/main/pad/pad.xml
```

Then in the Unraid web UI: open **Apps**, search for the app name (or browse to the **Private** category) → **Install** → form → **Apply**.

To remove later: delete the file. To update: re-run the `wget`.

### Route B — Docker tab sideload

If you don't have CA installed, or want to skip CA's listing UI:

```bash
ssh root@<your-tower>
mkdir -p /boot/config/plugins/dockerMan/templates-user
wget -O /boot/config/plugins/dockerMan/templates-user/my-pad.xml \
  https://raw.githubusercontent.com/PerpetualSoftware/unraid-templates/main/pad/pad.xml
```

Then in the Unraid web UI: **Docker** tab → **Add Container** → **Template** dropdown → pick the app → **Apply**.

Functionally identical container behavior; just skips the CA UX surface.

## Per-app docs

Each app directory carries its own README with app-specific notes (PUID/PGID guidance, env vars, backup pattern, etc.). Browse via the table above or the directory tree.

## Maintainer guide

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for:

- How to add a new template to this repo
- dockerMan SAVE format conformity rules (the rules that keep Squid happy)
- The Community Applications submission workflow (Validate, Scan, Submit via [ca.unraid.net/submit](https://ca.unraid.net/submit))
- Verification protocol for catching format drift before it ships

## Source repos

The runtime images themselves live in their respective product repos:

- **Pad** → [github.com/PerpetualSoftware/pad](https://github.com/PerpetualSoftware/pad)

This repo holds *only* the Unraid template metadata. Runtime bug reports go to the product repo's issue tracker.

## License

Apache 2.0 — see [`LICENSE`](LICENSE). Same license as the products themselves.
