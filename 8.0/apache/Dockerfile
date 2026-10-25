# PHP 8.0 is EOL upstream. Since the PHP2Shell disclosure (CVE-2026-109618 / CVE-2026-109619)
# the only maintained source of security fixes is packages.sury.org, so this image is built on
# plain Debian + Sury packages instead of the official php:8.0-apache image.
FROM debian:bookworm-slim

ARG PHP_VERSION=8.0
ENV DEBIAN_FRONTEND=noninteractive \
    PHP_VERSION=${PHP_VERSION} \
    PHP_INI_SCAN_DIR=:/usr/local/etc/php/conf.d \
    APACHE_RUN_USER=www-data \
    APACHE_RUN_GROUP=www-data \
    COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_MEMORY_LIMIT=-1

# Sury repository
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates curl lsb-release gnupg \
    && curl -sSLo /tmp/debsuryorg-archive-keyring.deb https://packages.sury.org/debsuryorg-archive-keyring.deb \
    && dpkg -i /tmp/debsuryorg-archive-keyring.deb && rm /tmp/debsuryorg-archive-keyring.deb \
    && echo "deb [signed-by=/usr/share/keyrings/debsuryorg-archive-keyring.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" \
        > /etc/apt/sources.list.d/php.list

# Apache + PHP + extensions (same set as the previous php:8.0-apache based image)
RUN apt-get update && apt-get install -y --no-install-recommends \
        apache2 \
        libapache2-mod-php${PHP_VERSION} \
        php${PHP_VERSION}-cli \
        php${PHP_VERSION}-bcmath \
        php${PHP_VERSION}-curl \
        php${PHP_VERSION}-gd \
        php${PHP_VERSION}-intl \
        php${PHP_VERSION}-mbstring \
        php${PHP_VERSION}-mysql \
        php${PHP_VERSION}-soap \
        php${PHP_VERSION}-sqlite3 \
        php${PHP_VERSION}-xml \
        php${PHP_VERSION}-xsl \
        php${PHP_VERSION}-zip \
        php${PHP_VERSION}-imagick \
        php${PHP_VERSION}-redis \
        php${PHP_VERSION}-uuid \
        php${PHP_VERSION}-http \
        php${PHP_VERSION}-raphf \
        php${PHP_VERSION}-opcache \
        php${PHP_VERSION}-amqp \
        php${PHP_VERSION}-mongodb \
        git unzip mc lynx cron \
    && rm -rf /var/lib/apt/lists/*

# Compatibility with the former official image: extra ini files dropped into the old path still work
RUN mkdir -p /usr/local/etc/php/conf.d

# PHP2Shell defence in depth: upload progress tracking is one of the two exploit paths
RUN printf 'session.upload_progress.enabled = 0\n' > /usr/local/etc/php/conf.d/99-php2shell-hardening.ini

# Apache: same behaviour as the official php:*-apache image
RUN a2dismod -f mpm_event mpm_worker >/dev/null 2>&1 || true \
    && a2enmod mpm_prefork rewrite \
    && sed -ri 's!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g; s!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g' /etc/apache2/sites-available/*.conf /etc/apache2/apache2.conf \
    && echo 'ServerName localhost' > /etc/apache2/conf-available/servername.conf && a2enconf servername \
    && mkdir -p /var/www/html /var/run/apache2 /var/lock/apache2 /var/log/apache2 \
    && chown -R www-data:www-data /var/www/html

# Same as docker-php.conf in the official image: honour .htaccess (front controller rewrites)
RUN printf '<Directory /var/www/>\n\tOptions -Indexes\n\tAllowOverride All\n</Directory>\nDirectoryIndex disabled\nDirectoryIndex index.php index.html\n' > /etc/apache2/conf-available/docker-php.conf \
    && a2enconf docker-php \
    && rm -f /var/www/html/index.html

# Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

## Crontab
COPY ./8.0/apache/app-cron /etc/cron.d/app-cron
RUN chmod 0644 /etc/cron.d/app-cron \
    && crontab /etc/cron.d/app-cron \
    && touch /var/log/cron.log

COPY ./8.0/apache/start-script /usr/local/bin/
RUN chmod 0755 /usr/local/bin/start-script
COPY ./shared/apache2-foreground /usr/local/bin/
COPY ./shared/docker-php-entrypoint /usr/local/bin/
RUN chmod 0755 /usr/local/bin/apache2-foreground /usr/local/bin/docker-php-entrypoint

RUN curl -sSL https://get.symfony.com/cli/installer | bash \
    && mv /root/.symfony*/bin/symfony /usr/local/bin/symfony

WORKDIR /var/www
EXPOSE 80
ENTRYPOINT ["docker-php-entrypoint"]
CMD ["bash", "/usr/local/bin/start-script"]
