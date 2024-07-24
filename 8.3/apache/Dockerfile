FROM php:8.3-apache

RUN apt-get update && \
    apt-get install vim -y && \
    apt-get install openssl -y && \
    apt-get install libssl-dev -y && \
    apt-get install wget -y && \
    apt-get install git -y && \
    apt-get install zip -y && \
    apt-get install libcurl4-openssl-dev -y && \
    apt-get install libonig-dev -y

RUN pecl install mongodb \
    &&  echo "extension=mongodb.so" > $PHP_INI_DIR/conf.d/mongo.ini
RUN docker-php-ext-install opcache
RUN docker-php-ext-install mbstring
RUN apt-get install -y libzip-dev && docker-php-ext-install zip
RUN docker-php-ext-install sockets
RUN apt-get install -y libxslt-dev \
    && docker-php-ext-install xsl
RUN docker-php-ext-install mysqli pdo_mysql zip bcmath intl
RUN apt-get install -y uuid-dev && pecl install uuid && docker-php-ext-enable uuid
RUN docker-php-ext-install exif
RUN apt-get update && apt-get install -y \
    libc-client-dev \
    libkrb5-dev
RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl
RUN docker-php-ext-install imap
RUN apt-get install -y libinotifytools0-dev && \
    pecl install inotify && \
    docker-php-ext-enable inotify

RUN apt-get autoremove -y && rm -rf /var/lib/apt/lists/*

ENV APACHE_RUN_USER=www-data
ENV APACHE_RUN_GROUP=www-data

RUN a2enmod rewrite

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER=1 COMPOSER_MEMORY_LIMIT=-1

## Crontab & ISync
RUN apt-get update && apt-get -y install cron isync
# Copy hello-cron file to the cron.d directory
COPY ./app-cron /etc/cron.d/app-cron
# Give execution rights on the cron job
RUN chmod 0644 /etc/cron.d/app-cron
# Apply cron job
RUN crontab /etc/cron.d/app-cron
# Create the log file to be able to run tail
RUN touch /var/log/cron.log

COPY ./start-script /usr/local/bin/
RUN chmod 0777 /usr/local/bin/start-script

RUN apt-get install -y supervisor
RUN mkdir -p /var/log/supervisor
COPY ./supervisor/supervisord.conf /etc/supervisor/supervisord.conf
COPY ./supervisor/conf.d /etc/supervisor/conf.d

WORKDIR /var/www
CMD ["/usr/bin/supervisord"]
