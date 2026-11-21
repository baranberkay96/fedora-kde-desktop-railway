# Fedora 44 KDE Plasma Desktop in Your Browser: composer spec

Listing: **Fedora 44 KDE Plasma Desktop in Your Browser**, "Fedora 44 KDE Plasma desktop in a browser tab with a persistent home.". Category Other. Icon `https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/linux.svg`.

## Service: Desktop

- Source: repo `baranberkay96/fedora-kde-desktop-railway`, root directory `/services/desktop` (Dockerfile).
- Base: `lscr.io/linuxserver/webtop:fedora-kde-e6db4cc0-ls306@sha256:7701fecc2c01d70db77b1f04fbe84563d757752727a6cb919ee7ba894174a7ab` (built 2026-11-17, 1.90 GB compressed).
- Matching software-rendering layer: `ghcr.io/linuxserver/selkies-layers:amd64-fedora44-kwin@sha256:fd08eb7e43f41f7c35a368c43585e7f8f8d6595746fd0523c65d3352dcba3ed3` (KWin 6.8.3, checked 2026-11-21).
- Public domain, target port 3000, healthcheck `/healthz` (600 s), restart ON_FAILURE ×10, 1 replica.
- Volume `/config`.
- Variables: `PASSWORD` = `${{secret(32)}}`, `CUSTOM_USER` = `fedora`, `PORT` = `3000`, `TZ` = `Etc/UTC`, `TITLE` = `Fedora 44 KDE Plasma Desktop`, `DNF_PACKAGES` = `empty`, `RAILWAY_DEPLOYMENT_DRAINING_SECONDS` = `15`.

## Improvements over market templates

- Login always on (basic auth enabled at build time), generated password; an empty, short or overlong password locks the login instead of opening the desktop or failing the deploy (STANDARD 8.29).
- Works under Railway's podman-init (the image's s6 `/init` refuses to run when it is not PID 1): `railway-start` brings up the s6 tree itself.
- Origin check against cross-site WebSocket/upload requests, `SELKIES_ALLOWED_ORIGINS` same-origin, anti-framing headers, `/healthz` that checks Selkies.
- The base bundled a patched KWin 6.8.2 library alongside a stock 6.8.3 executable. The template overlays the official patched 6.8.3 library and screencast plugin; the build checks its software-rendering marker. Actual Plasma wallpaper and panel verified over authenticated Selkies JPEG frames.
- Pinned release and digest; nginx workers capped at 2; Docker-in-Docker off; packages reinstalled after redeploys from `DNF_PACKAGES`.

## Resource estimate

| Service | Idle RAM | Typical | CPU |
|---|---|---|---|
| Desktop | about 912 MiB including cache while streaming under a 953 MiB cgroup cap; no OOM kills | 2 GB or more with Chromium | 0.1 vCPU idle, 1–2 vCPU while the screen changes |

About $9–15/month idle at Railway prices plus the volume ($0.15/GB-month); more while the desktop is in use. Minimum plan: Trial (1 GB) verified for the bare desktop; Hobby with 2 GB or more recommended for applications. Free (0.5 GB) is too small.

## Measured verification (2026-11-21)

- amd64 Docker build passed, including a check for the official software-rendering KWin marker. Independent code review approved the overlay.
- Railway deployment `c4b5b989-5b33-4d58-ae59-912db236a910` from `92cd3f2`: SUCCESS, real Plasma wallpaper, panel and clock visible over authenticated Selkies JPEG.
- Plan simulation: `memory.max=999997440`, `memory.current=956084224`, `oom_kill=0`; bare desktop and streaming work at the Trial cap.
- At 2 GB: `PASSWORD=short` deployment stayed healthy; anonymous and weak-password requests returned 401. `/config/railway-persistence-proof.txt` survived redeploy.
- `DNF_PACKAGES=htop` restored `htop-3.4.1-3.fc44.x86_64`; logs reported success and removed 108 MiB of package cache.

## Verification checklist

- `/healthz` 200 without login; `/` 401 without and 200 with the generated credential.
- The desktop session starts and streams; a file on `~/Desktop` survives a redeploy.
- `PASSWORD=short` → deploy succeeds, login locked, log explains.
- `DNF_PACKAGES` with a small package is installed after start (`/var/log/railway-desktop/pkg-restore.log`).

## Open questions

- Trademark: Fedora is a trademark of Red Hat, Inc. The Fedora name is used only to describe the software in the image; the listing uses a generic Linux icon, not the Fedora logo (whose use needs permission under the Fedora trademark guidelines).
