FROM php:8.4-fpm-bookworm

ARG APCU_VERSION=5.1.28
ARG GNUPG_VERSION=1.5.4
ARG IMAP_VERSION=1.0.3
ARG MAILPARSE_VERSION=3.2.0
ARG MONGODB_VERSION=2.5.2
ARG MSGPACK_VERSION=3.0.1
ARG REDIS_VERSION=6.3.0
ARG SSH2_VERSION=1.5.0
ARG SWOOLE_VERSION=6.2.2

LABEL org.opencontainers.image.source="https://github.com/KongHack/php-image" \
      org.opencontainers.image.description="KongHack PHP 8.4 FPM base image" \
      org.opencontainers.image.licenses="MIT"

ENV PHP_PEAR_PHP_BIN="" \
    PHP_PEAR_INSTALL_DIR="" \
    PHP_PEAR_SYSCONF_DIR="" \
    PHP_PEAR_TEMP_DIR=""

# Debian packages intentionally track repository security updates.
# hadolint ignore=DL3008
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      git \
      mariadb-client \
      unzip \
 && rm -rf /var/lib/apt/lists/*

# PHPIZE_DEPS must be word-split. The final smoke test catches missing runtime
# libraries if any earlier command in the dependency-discovery pipeline fails.
# hadolint ignore=DL3008,DL4006,SC2086
RUN set -eux; \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      $PHPIZE_DEPS \
      libc-client-dev \
      libbrotli-dev \
      libcurl4-openssl-dev \
      libfreetype6-dev \
      libgpgme-dev \
      libicu-dev \
      libjpeg62-turbo-dev \
      libkrb5-dev \
      libonig-dev \
      libpng-dev \
      libssh2-1-dev \
      libssl-dev \
      libwebp-dev \
      libxml2-dev \
      libzip-dev; \
    docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp; \
    docker-php-ext-install -j"$(nproc)" \
      bcmath \
      curl \
      exif \
      fileinfo \
      gd \
      intl \
      mbstring \
      pcntl \
      pdo \
      pdo_mysql \
      posix \
      sockets \
      xml \
      zip; \
    pecl channel-update pecl.php.net; \
    export MAKEFLAGS="-j$(nproc)"; \
    pecl install \
      "redis-${REDIS_VERSION}" \
      "apcu-${APCU_VERSION}" \
      "ssh2-${SSH2_VERSION}" \
      "gnupg-${GNUPG_VERSION}" \
      "mailparse-${MAILPARSE_VERSION}" \
      "msgpack-${MSGPACK_VERSION}" \
      "swoole-${SWOOLE_VERSION}" \
      "mongodb-${MONGODB_VERSION}" \
      "imap-${IMAP_VERSION}"; \
    docker-php-ext-enable \
      apcu \
      gnupg \
      imap \
      mailparse \
      mongodb \
      msgpack \
      redis \
      ssh2 \
      swoole; \
    apt-mark auto '.*' > /dev/null; \
    if [ -n "$savedAptMark" ]; then apt-mark manual $savedAptMark; fi; \
    find /usr/local/lib/php/extensions -type f -name '*.so' -exec ldd '{}' ';' \
      | awk '/=>/ { so = $(NF - 1); if (index(so, "/usr/local/") == 1) next; gsub("^/(usr/)?", "", so); print so }' \
      | sort -u \
      | xargs -r dpkg-query --search \
      | cut -d: -f1 \
      | sort -u \
      | xargs -r apt-mark manual; \
    apt-get purge -y --auto-remove; \
    rm -rf /tmp/pear /var/lib/apt/lists/*

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

COPY php-8.4/php.ini "$PHP_INI_DIR/conf.d/zz-konghack.ini"
COPY php-8.4/www.conf /usr/local/etc/php-fpm.d/zz-konghack.conf
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Dollar-prefixed expressions below are PHP, not shell expansions.
# hadolint ignore=SC2016
RUN php -r 'foreach (["apcu", "bcmath", "curl", "exif", "fileinfo", "gd", "gnupg", "imap", "intl", "mailparse", "mbstring", "mongodb", "msgpack", "pcntl", "pdo_mysql", "posix", "redis", "sockets", "ssh2", "swoole", "xml", "zip"] as $extension) { if (!extension_loaded($extension)) { fwrite(STDERR, "Missing extension: {$extension}\n"); exit(1); } }' \
 && php -r 'if (ini_get("session.save_handler") !== "files") { fwrite(STDERR, "Unexpected session handler\n"); exit(1); }' \
 && php-fpm -t \
 && composer --version

EXPOSE 9000

CMD ["php-fpm"]
