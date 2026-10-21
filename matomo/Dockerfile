# syntax=docker/dockerfile:1.5.1
ARG TAG=7.0.21
FROM islandora/nginx:${TAG}

ARG TARGETARCH

ARG MATOMO_VERSION="5.3.2"
ARG MATOMO_FILE="${MATOMO_VERSION}.tar.gz"
ARG MATOMO_URL="https://builds.matomo.org/matomo-${MATOMO_FILE}"
ARG MATOMO_SHA256="ae7e4baf6052ac68ad235e8c2e53fdd739d23e6d80b1dea3d2a23c37ed5cf8ba"

ARG EXTRA_TOOLS_VERSION="5.1.2"
ARG EXTRA_TOOLS_FILE="${EXTRA_TOOLS_VERSION}.tar.gz"
ARG EXTRA_TOOLS_URL="https://github.com/digitalist-se/extratools/archive/refs/tags/${EXTRA_TOOLS_FILE}"
ARG EXTRA_TOOLS_SHA256="5995c7618772d676252443a3b8c3caf3b93748a7ef52a54b19ea16d20bd8594a"

EXPOSE 8000

WORKDIR /var/www/matomo

# Composer require statements are for supporting the Extra Tools plugin (constraints from its composer.json).
# --update-no-dev: Matomo's require-dev packages (php_codesniffer, xhprof, ...) don't belong in a production image.
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
    composer require -d /var/www/matomo --no-interaction --update-no-dev symfony/yaml:~5.4.21 symfony/process:^5.4 && \
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

# fastcgi_pass in default.conf.tmpl must match the base image's FPM socket. Fail the build if it doesn't.
RUN grep -q 'fastcgi_pass unix:/run/php-fpm84/php-fpm84.sock;' /etc/confd/templates/default.conf.tmpl && \
    grep -q '^listen = php-fpm84.sock$' /etc/confd/templates/www.conf.tmpl

# The MariaDB client in buildkit >= 5 (11.x) requires SSL by default, and our mariadb service
# (islandora/mariadb:2.0.8) has SSL disabled. Without this, execute-sql-file.sh / create-database.sh /
# wait-for-database.sh / mysqldump fail, and the matomo-setup and install oneshots fail with them.
RUN printf '[client]\nskip-ssl\n' > /etc/my.cnf.d/client.cnf

RUN chown -R nginx:nginx /var/www
