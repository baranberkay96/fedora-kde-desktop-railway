# syntax=docker/dockerfile:1
#
# Fedora 44 with KDE Plasma (Wayland), in the browser, for Railway.
#
# Base: linuxserver.io webtop, release fedora-kde-e6db4cc0-ls306 (built
# 2026-11-17 from the fedora-kde branch of linuxserver/docker-webtop, on
# baseimage-selkies:fedora44). The digest is the multi-arch index, identical
# on lscr.io, ghcr.io and Docker Hub. To upgrade, replace tag and digest
# together with a newer "fedora-kde-<hash>-ls<n>" release and redeploy.
FROM lscr.io/linuxserver/webtop:fedora-kde-e6db4cc0-ls306@sha256:7701fecc2c01d70db77b1f04fbe84563d757752727a6cb919ee7ba894174a7ab

# The base contains Fedora KWin 6.8.3 but its software-rendering layer is
# 6.8.2 and is never loaded. Use the matching official patched 6.8.3 layer.
COPY --from=ghcr.io/linuxserver/selkies-layers:amd64-fedora44-kwin@sha256:fd08eb7e43f41f7c35a368c43585e7f8f8d6595746fd0523c65d3352dcba3ed3 / /
RUN grep -aq "Rendering in software on" /usr/lib64/libkwin.so.6.8.3

# Image defaults for Railway. Variables set on the Railway service win.
#  - PUID/PGID 1000: fixed owner of the /config volume. Do not change them
#    after the first boot, because files already on the volume keep the old ids.
#  - START_DOCKER=false: Railway gives no privileged containers, so the
#    bundled Docker daemon can never run there.
#  - SELKIES_ALLOWED_ORIGINS empty: the image ships "*", which lets any
#    website open the desktop's WebSocket with the browser's cached login.
#    Empty is Selkies' own default: same-origin only.
#  - SELKIES_ENABLE_BASIC_AUTH=false: nginx does the login. Selkies listens
#    on loopback only.
#  - PIXELFLUX_WAYLAND=true: KDE runs as a Wayland session, as upstream.
ENV PUID=1000 \
    PGID=1000 \
    START_DOCKER=false \
    SELKIES_ALLOWED_ORIGINS="" \
    SELKIES_ENABLE_BASIC_AUTH=false \
    PIXELFLUX_WAYLAND=true \
    TITLE="Fedora 44 KDE Plasma Desktop" \
    CUSTOM_USER=fedora \
    PORT=3000

COPY patch-image.sh /tmp/railway-desktop/patch-image.sh
COPY nginx-origin-map.conf nginx-server-hardening.conf /tmp/railway-desktop/
COPY railway-start.sh /usr/local/bin/railway-start
COPY pkg-restore.sh /usr/local/bin/railway-pkg-restore

# Build-time edits of the image's nginx files. The script checks every
# anchor it edits and fails the build if the upstream files have changed.
RUN bash /tmp/railway-desktop/patch-image.sh \
 && rm -rf /tmp/railway-desktop \
 && chmod 0755 /usr/local/bin/railway-start /usr/local/bin/railway-pkg-restore

EXPOSE 3000

# Railway runs its own init (podman-init) as PID 1, and the image's /init
# (s6-overlay 3.2.1.0) refuses to start unless it is PID 1. railway-start
# brings up the same s6 supervision tree without that check. catatonit
# (part of the linuxserver.io base image) reaps zombies and forwards signals,
# as a child subreaper when it is not PID 1 itself.
ENTRYPOINT ["/usr/bin/catatonit", "--", "/usr/local/bin/railway-start"]
