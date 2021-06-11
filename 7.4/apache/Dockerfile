FROM php:7.4-apache

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data

RUN a2enmod rewrite

# Install php extensions
RUN apt-get update && apt-get install -y \
        git \
        unzip \
        g++ \
        libzip-dev \
        libssl-dev \
    && pecl -q install \
        zip \
    && docker-php-ext-enable zip
RUN apt-get install -y mc lynx
RUN apt-get update -y && apt-get install -y \
    libwebp-dev libjpeg62-turbo-dev libpng-dev libxpm-dev \
    libfreetype6-dev libonig-dev zlib1g-dev
RUN docker-php-ext-install mbstring
RUN apt-get install -y libzip-dev
RUN docker-php-ext-install zip
RUN docker-php-ext-configure gd --with-webp --with-jpeg --with-xpm --with-freetype
RUN docker-php-ext-install gd
RUN docker-php-ext-install sockets
RUN apt-get install -y libxslt-dev \
    && docker-php-ext-install xsl
RUN docker-php-ext-install mysqli pdo_mysql zip bcmath intl
RUN apt-get install -y uuid-dev && pecl install uuid && docker-php-ext-enable uuid
RUN docker-php-ext-install exif
RUN apt-get install -y libmagickwand-dev --no-install-recommends && rm -rf /var/lib/apt/lists/*
RUN mkdir -p /usr/src/php/ext/imagick; \
    curl -fsSL https://github.com/Imagick/imagick/archive/06116aa24b76edaf6b1693198f79e6c295eda8a9.tar.gz | tar xvz -C "/usr/src/php/ext/imagick" --strip 1; \
    docker-php-ext-install imagick
RUN pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis
RUN docker-php-ext-install soap
RUN docker-php-ext-install simplexml

COPY custom.ini /usr/local/etc/php/conf.d/custom.ini

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER=1 COMPOSER_MEMORY_LIMIT=-1
RUN composer self-update --2