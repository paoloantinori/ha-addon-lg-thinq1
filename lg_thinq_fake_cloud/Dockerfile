# HAOS app image for the LG ThinQ1 fake-cloud (store-repo build).
# This Dockerfile lives in the lightweight store repo. It clones the main server repo at a
# pinned commit and builds it, so the store repo stays small (no vendored server copy).
# Pattern follows The-sultan/hassio-rethink-addon (which clones anszom/rethink the same way).
#
# To update the add-on to a newer server version: bump MAIN_COMMIT below and rebuild.
FROM ghcr.io/home-assistant/base:latest

# The main repo + the exact commit (already pushed to origin) this add-on builds from.
ARG MAIN_REPO=https://github.com/paoloantinori/local_smarthinq1.git
ARG MAIN_COMMIT=9e5c5361038ddf6e2beac3eb9d28c04a4fc765f5

RUN apk add --no-cache git python3 py3-pip openssl jq

# Clone the server repo at the pinned commit into /app.
RUN git clone "${MAIN_REPO}" /app && cd /app && git checkout "${MAIN_COMMIT}"

WORKDIR /app

# The add-on's run.sh is shipped in THIS store repo (next to this Dockerfile), not in the main
# repo, so copy it over the main repo's copy and place it at /run.sh for s6 (the HAOS base
# invokes /run.sh at the image root as the entrypoint).
COPY run.sh /run.sh
RUN cp /run.sh /app/deploy/haos-addon/run.sh && chmod a+x /run.sh

# paho-mqtt is the only non-stdlib dependency (the MQTT bridge).
RUN pip install --no-cache-dir --break-system-packages paho-mqtt

# The unified HA base does NOT auto-run /run.sh: register it as a legacy s6
# service (legacy-services oneshot reads /etc/services.d/*/run). Without this
# the container boots the s6 tree and then idles with no app process.
RUN mkdir -p /etc/services.d/lg-fake-cloud \
    && printf '#!/bin/sh\nexec /run.sh\n' > /etc/services.d/lg-fake-cloud/run \
    && chmod +x /etc/services.d/lg-fake-cloud/run
