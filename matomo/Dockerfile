# syntax=docker/dockerfile:1.5.1
FROM nginx

ARG TARGETARCH

ARG MATOMO_VERSION="5.0.3"
ARG MATOMO_FILE="${MATOMO_VERSION}.tar.gz"
ARG MATOMO_URL="https://builds.matomo.org/matomo-${MATOMO_FILE}"
ARG MATOMO_SHA256="cd656ee7df4b29ac453456c6b9708d8264093ac2c99d6669d2830f31624cd626"

ARG EXTRA_TOOLS_VERSION="5.0.1"
ARG EXTRA_TOOLS_FILE="${EXTRA_TOOLS_VERSION}.tar.gz"
ARG EXTRA_TOOLS_URL="https://github.com/digitalist-se/extratools/archive/refs/tags/${EXTRA_TOOLS_FILE}"
ARG EXTRA_TOOLS_SHA256="e76592e839a6aa0c73189790bed7e0032281e58a6b5b648278844b664367267c"

EXPOSE 8000

WORKDIR /var/www/matomo

# Composer require statements are for supporting the Extra Tools plugin.
#
# Platform agnostic does not require arch specific identifier.
RUN --mount=type=cache,id=matomo-downloads-${TARGETARCH},sharing=locked,target=/opt/downloads \
    download.sh \
        --url "${MATOMO_URL}" \
        --sha256 "${MATOMO_SHA256}" \
        --strip \
        --dest /var/www/matomo \
    && \
    wget --directory-prefix=/var/www/matomo "https://raw.githubusercontent.com/matomo-org/matomo/${MATOMO_VERSION}/composer.json" && \
    wget --directory-prefix=/var/www/matomo "https://raw.githubusercontent.com/matomo-org/matomo/${MATOMO_VERSION}/composer.lock" && \
    composer require -d /var/www/matomo symfony/yaml:~2.6.0 && \
    composer require -d /var/www/matomo symfony/process:^5.4 && \
    composer install -d /var/www/matomo && \
    download.sh \
        --url "${EXTRA_TOOLS_URL}" \
        --sha256 "${EXTRA_TOOLS_SHA256}" \
        --strip \
        --dest /var/www/matomo/plugins/ExtraTools && \
    cleanup.sh

# The driver is given explicitly to prevent accidentially overriding as Matomo
# only supports MySQL.
ENV \
    MATOMO_ASSUME_SECURE_PROTOCOL=1 \
    MATOMO_DB_DRIVER=mysql \
    MATOMO_DB_NAME=matomo \
    MATOMO_DB_PASSWORD=password \
    MATOMO_DB_USER=matomo \
    MATOMO_DEFAULT_HOST=https://islandora.traefik.me \
    MATOMO_DEFAULT_NAME=Islandora \
    MATOMO_DEFAULT_TIMEZONE=America/Halifax \
    MATOMO_FORCE_SSL=1 \
    MATOMO_PROXY_CLIENT_HEADERS=HTTP_X_FORWARDED_FOR \
    MATOMO_PROXY_HOST_HEADERS=HTTP_X_FORWARDED_HOST \
    MATOMO_PROXY_URI_HEADER=1 \
    MATOMO_USER_EMAIL=admin@example.org \
    MATOMO_USER_NAME=admin \
    MATOMO_USER_PASS=password

COPY --link rootfs /

RUN chown -R nginx:nginx /var/www
