ARG BUILD_FROM=ghcr.io/hassio-addons/base/aarch64:21.0.2
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base packages. Kept in its own layer, deliberately not combined with
# the Tailscale binary download below: this layer only changes when the apk
# package list changes, so it stays cached across the daily version-bump
# commits that only touch TAILSCALE_VERSION, instead of reinstalling every
# package from scratch on every update.
# Deliberately unpinned (not the usual hassio-addons convention): exact apk
# version pins go stale on Alpine's own repo rotation schedule, independent
# of anything in this repo, and have already broken a build once. Accepting
# "whatever apk add pulls today" instead, low risk for these packages.
# hadolint ignore=DL3018
RUN \
    apk add --no-cache \
        bind-tools \
        dnsmasq \
        ethtool \
        ipcalc \
        iproute2 \
        iptables \
        nginx \
        coreutils \
        networkmanager \
    \
    && ln -sf /sbin/xtables-nft-multi /sbin/ip6tables \
    && ln -sf /sbin/xtables-nft-multi /sbin/iptables \
    \
    && rm -f -r \
        /usr/lib/NetworkManager/dispatcher.d

# Fetch the Tailscale binary. Separate layer from the packages above, so a
# version bump only re-downloads the binary, not the whole apk install.
ARG BUILD_ARCH=aarch64
ARG TAILSCALE_VERSION="v1.102.4"
RUN \
    if [ "${BUILD_ARCH}" = "aarch64" ]; then ARCH="arm64"; fi \
    && if [ "${BUILD_ARCH}" = "amd64" ]; then ARCH="amd64"; fi \
    \
    && curl -L -s --fail --retry 3 --retry-delay 5 --retry-connrefused \
        "https://pkgs.tailscale.com/stable/tailscale_${TAILSCALE_VERSION#v}_${ARCH}.tgz" \
        | tar zxvf - -C /opt/ --strip-components 1 \
    \
    && rm -f -r \
        /opt/systemd

# Copy root filesystem
COPY rootfs /

HEALTHCHECK \
    CMD healthcheck

# S6 Overlay stage 2 hook
ENV S6_STAGE2_HOOK=/etc/s6-overlay/scripts/stage2_hook.sh

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <opensource@frenck.dev>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Apps" \
    org.opencontainers.image.authors="Franck Nijhof <opensource@frenck.dev>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://frenck.dev/home-assistant-apps" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
