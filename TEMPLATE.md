# Fedora 44 KDE Plasma Desktop in Your Browser: composer spec

Listing: **Fedora 44 KDE Plasma Desktop in Your Browser**, "Fedora 44 KDE Plasma desktop in a browser tab with a persistent home.". Category Other. Icon `https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/linux.svg`.

## Service: Desktop

- Source: repo `baranberkay96/fedora-kde-desktop-railway`, root directory `/services/desktop` (Dockerfile).
- Base: `lscr.io/linuxserver/webtop:fedora-kde-e6db4cc0-ls306@sha256:7701fecc2c01d70db77b1f04fbe84563d757752727a6cb919ee7ba894174a7ab` (built 2026-11-17, 1.90 GB compressed).
- Public domain, target port 3000, healthcheck `/healthz` (600 s), restart ON_FAILURE ×10, 1 replica.
- Volume `/config`.
- Variables: `PASSWORD` = `${{secret(32)}}`, `CUSTOM_USER` = `fedora`, `PORT` = `3000`, `TZ` = `Etc/UTC`, `TITLE` = `Fedora 44 KDE Plasma Desktop`, `DNF_PACKAGES` = `empty`, `RAILWAY_DEPLOYMENT_DRAINING_SECONDS` = `15`.

## Improvements over market templates

- Login always on (basic auth enabled at build time), generated password; an empty, short or overlong password locks the login instead of opening the desktop or failing the deploy (STANDARD 8.29).
- Works under Railway's podman-init (the image's s6 `/init` refuses to run when it is not PID 1): `railway-start` brings up the s6 tree itself.
- Origin check against cross-site WebSocket/upload requests, `SELKIES_ALLOWED_ORIGINS` same-origin, anti-framing headers, `/healthz` that checks Selkies.
- Pinned release and digest; nginx workers capped at 2; Docker-in-Docker off; packages reinstalled after redeploys from `DNF_PACKAGES`.

## Resource estimate

| Service | Idle RAM | Typical | CPU |
|---|---|---|---|
| Desktop | about 0.61 GB of RAM idle and 0.67 GB with a browser session streaming (not yet tested under a 1 GB limit, so Trial is likely but unconfirmed) | 1–2 GB with Chromium | 0.1 vCPU idle, 1–2 vCPU while the screen changes |

About $6–10/month idle at Railway prices plus the volume ($0.15/GB-month); more while the desktop is in use. Minimum plan: Trial (1 GB); Free (0.5 GB) is too small.

## Verification checklist

- `/healthz` 200 without login; `/` 401 without and 200 with the generated credential.
- The desktop session starts and streams; a file on `~/Desktop` survives a redeploy.
- `PASSWORD=short` → deploy succeeds, login locked, log explains.
- `DNF_PACKAGES` with a small package is installed after start (`/var/log/railway-desktop/pkg-restore.log`).

## Open questions

- Trademark: Fedora is a trademark of Red Hat, Inc. The Fedora name is used only to describe the software in the image; the listing uses a generic Linux icon, not the Fedora logo (whose use needs permission under the Fedora trademark guidelines).
