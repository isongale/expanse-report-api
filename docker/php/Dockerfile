# Development image for the Laravel application (PHP-FPM).
# The same image runs the "app" service and, later, the queue worker.
FROM php:8.5-fpm

# Host user IDs, so that files written to the bind mount belong to the developer.
ARG UID=1000
ARG GID=1000

# Build dependencies for the extensions below (headers only needed at compile time
# are kept: this is a dev image, size is not the priority).
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        git \
        unzip \
        libicu-dev \
        libzip-dev \
        libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Extensions bundled with PHP: compiled from the PHP source tree.
# OPcache is not listed because it is always built in since PHP 8.5.
RUN docker-php-ext-install -j"$(nproc)" \
        pdo_mysql \
        pcntl \
        intl \
        zip \
        bcmath

# Extensions distributed through PECL: client drivers for Redis and MongoDB.
RUN pecl install redis mongodb \
    && docker-php-ext-enable redis mongodb \
    && rm -rf /tmp/pear

# Development php.ini shipped with the image (errors displayed, assertions on).
RUN cp "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

# Composer binary taken from the official image instead of running its installer.
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Unprivileged user matching the host UID/GID.
RUN groupadd --gid "$GID" app \
    && useradd --uid "$UID" --gid "$GID" --create-home --shell /bin/bash app

USER app
WORKDIR /var/www/html

# php-fpm listens on 9000 (FastCGI); CMD is inherited from the base image.
EXPOSE 9000
