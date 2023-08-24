from php:fpm-alpine

LABEL version="1.0"
LABEL maintainer="Pukar Sharma <sharmapukar217@gmail.com>"

ARG SUPPORTED_LOCALES="en_US.UTF-8"

ENV AKAUNTING_VERSION=3.0.16 \
    COMPOSER_ALLOW_SUPERUSER=1

# install required dependencies
RUN apk --no-cache add git python3 nodejs npm icu-dev zlib-dev libpng-dev libzip-dev

# install php extensions
RUN docker-php-ext-install bcmath intl gd zip 

# configure php extensions
# RUN docker-php-ext-configure opcache --enable-opcache

# setup working dir
WORKDIR /var/www/html

# install composer
COPY --from=composer:2.4 /usr/bin/composer /usr/bin/composer

# setup src
RUN git clone https://github.com/akaunting/akaunting.git . \
    && chown -R www-data:www-data /var/www/html

# copy env vars
COPY .env .

# run configuration
USER www-data

# run building
RUN composer prod \
    && npm i && npm run prod
