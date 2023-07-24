FROM debian:latest

LABEL maintainer="https://github.com/factoriotools/factorio-docker"

ARG USER=factorio
ARG GROUP=factorio
ARG PUID=845
ARG PGID=845

# version checksum of the archive to download
ARG VERSION
ARG SHA256

# number of retries that curl will use when pulling the headless server tarball
ARG CURL_RETRIES=8

ENV PORT=34197 \
    RCON_PORT=27015 \
    VERSION=${VERSION} \
    SHA256=${SHA256} \
    SAVES=/factorio/saves \
    CONFIG=/factorio/config \
    MODS=/factorio/mods \
    SCENARIOS=/factorio/scenarios \
    SCRIPTOUTPUT=/factorio/script-output \
    PUID="$PUID" \
    PGID="$PGID"

SHELL ["/bin/bash", "-eo", "pipefail", "-c"]
RUN set -ox pipefail \
    && if [[ "${VERSION}" == "" ]]; then \
        echo "build-arg VERSION is required" \
        && exit 1; \
    fi \
    && if [[ "${SHA256}" == "" ]]; then \
        echo "build-arg SHA256 is required" \
        && exit 1; \
    fi \
    && archive="/tmp/factorio_headless_x64_$VERSION.tar.xz" \
    && mkdir -p /opt /factorio \
    && dpkg --add-architecture amd64 \
    && apt -qq update \
    && apt -qq install -y binutils curl file gettext gosu jq libc6:amd64 pwgen xz-utils \
    && curl -sSL "https://www.factorio.com/get-download/$VERSION/headless/linux64" -o "$archive" --retry $CURL_RETRIES\
    && echo "$SHA256  $archive" | sha256sum -c \
    || (sha256sum "$archive" && file "$archive" && exit 1) \
    && tar xf "$archive" --directory /opt \
    && chmod ugo=rwx /opt/factorio \
    && rm "$archive" \
    && ln -s "$SCENARIOS" /opt/factorio/scenarios \
    && ln -s "$SAVES" /opt/factorio/saves \
    && mkdir -p /opt/factorio/config/ \
    && addgroup -gid "$PGID" --system "$GROUP" \
    && adduser -uid "$PUID" --ingroup "$GROUP" --shell /bin/sh --system --disabled-password --no-create-home "$USER" \
    && chown -R "$USER":"$GROUP" /opt/factorio /factorio

COPY files/*.sh /
COPY files/config.ini /opt/factorio/config/config.ini

VOLUME /factorio
EXPOSE $PORT/udp $RCON_PORT/tcp
ENTRYPOINT ["/docker-entrypoint.sh"]
