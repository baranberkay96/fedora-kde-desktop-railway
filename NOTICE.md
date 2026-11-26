# NOTICE: Fedora 44 KDE Plasma Desktop in Your Browser template

## Base image and its components

The service is built `FROM ghcr.io/linuxserver/webtop:fedora-kde-e6db4cc0-ls306@sha256:7701fecc2c01d70db77b1f04fbe84563d757752727a6cb919ee7ba894174a7ab` (built 2026-11-17; multi-arch index digest checked on Docker Hub 2026-11-20, 1.90 GB compressed for amd64, below the 4 GB Trial/Free image limit). The base is overlaid with the official ABI-matched KWin software-rendering layer listed below. This repository contains no copy of its source files.

| Component | Source | License | How it is used |
|---|---|---|---|
| linuxserver.io `webtop`, release `fedora-kde-e6db4cc0-ls306` | https://github.com/linuxserver/docker-webtop/tree/fedora-kde (branch head `75d31499e17a` checked 2026-11-20) | GPL-3.0 | Base image (`FROM`), unmodified except for the build-time edits listed below |
| linuxserver.io baseimage-selkies (`baseimage-selkies:fedora44`) | https://github.com/linuxserver/docker-baseimage-selkies | GPL-3.0 | Layers of the base image: the s6 service definitions, `init-nginx`, `/defaults/default.conf` |
| linuxserver.io KWin layer, `amd64-fedora44-kwin@sha256:fd08eb7e43f41f7c35a368c43585e7f8f8d6595746fd0523c65d3352dcba3ed3` | https://github.com/linuxserver/selkies-layers (`Dockerfile.fedora.kwin`, `patches/fedora/44/kwin`) | KWin GPL-2.0-or-later; layer build scripts GPL-3.0 | Replaces `libkwin.so.6.8.3` and the matching screencast plugin; upstream source and patches remain available at that repository |
| Selkies | https://github.com/selkies-project/selkies | MPL-2.0 | Streaming server and web client, in the base image |
| s6-overlay 3.2.1.0 | https://github.com/just-containers/s6-overlay, https://skarnet.org/software/ | ISC | Process supervision. `railway-start` calls `s6-svscan`, `s6-rc-compile`, `s6-rc-init` and `s6-rc` from the image |
| catatonit | https://github.com/openSUSE/catatonit (distribution package in the base image) | GPL-2.0-or-later | Container init and subreaper (`ENTRYPOINT`) |
| Fedora 44 packages: KDE Plasma, Dolphin, Konsole, KWrite, Discover, PipeWire, Mesa, nginx, Chromium and others | https://packages.fedoraproject.org/ | Each package's own license (see the package metadata in the image) | In the base image |

### GPL notes

- The GPL parts of the base image stay under their licenses. Their complete corresponding source is published by their authors at the repositories above, at the git revision named in the image label.
- **Modified files:** at build time, `services/desktop/patch-image.sh` edits `/defaults/default.conf` (GPL-3.0, from docker-baseimage-selkies: basic auth switched on, an Origin check, anti-framing headers and a `/healthz` location) and `/etc/nginx/nginx.conf` (2 worker processes, error log on stderr). The script and the inserted snippets (`nginx-origin-map.conf`, `nginx-server-hardening.conf`) are the complete record of these changes.  Anyone who distributes a built image of this template should keep them available with it.
- The files written for this template (`Dockerfile`, `railway-start.sh`, `pkg-restore.sh`, `patch-image.sh`, the two nginx snippets and the documentation) are offered under GPL-3.0-or-later.
- No source file of linuxserver.io, Selkies or s6-overlay was copied into this repository. `railway-start` re-implements, in its own code, the order of steps that s6-overlay's `preinit`, `stage0` and `rc.init` perform, and then calls the image's own s6 tools and service definitions.

## Upstream files checked (2026-11-20 and 2026-11-21)

- `Dockerfile` and `root/` tree of the `fedora-kde` branch of linuxserver/docker-webtop.
- `Dockerfile` and `root/etc/s6-overlay/s6-rc.d/init-nginx/run` of the `fedora44` branch of linuxserver/docker-baseimage-selkies (nginx site path and include line that `patch-image.sh` relies on).
- `linuxserver/selkies-layers/Dockerfile.fedora.kwin` and software-shm patches; pinned layer inspected to contain only the two matching KWin artifacts.
- Docker Hub tag list of `linuxserver/webtop` for the release tag, digest and compressed size.

## Market templates reviewed

`linux-desktop-debian`, `linux-desktop-alpine` and `linux-desktop-lxqt` ("Linux Desktop | ... in the Cloud"): listings read for comparison only. Nothing copied. The template is derived from our own Debian 13 KDE Plasma, Ubuntu 26.04 XFCE and Kali Linux KDE desktop templates.

Product names belong to their owners. This is a community template, not affiliated with linuxserver.io, the Selkies project, the distribution's publishers or Railway. Fedora is a trademark of Red Hat, Inc. The Fedora name is used only to describe the software in the image; the listing uses a generic Linux icon, not the Fedora logo (whose use needs permission under the Fedora trademark guidelines).
