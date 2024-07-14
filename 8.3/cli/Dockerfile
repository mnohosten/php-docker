FROM openswoole/swoole:php8.3

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER=1 COMPOSER_MEMORY_LIMIT=-1

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

RUN apt-get autoremove -y && rm -rf /var/lib/apt/lists/*
