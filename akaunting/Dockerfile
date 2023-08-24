from php:fpm-alpine

LABEL version="1.0"
LABEL maintainer="Pukar Sharma <sharmapukar217@gmail.com>"

ARG SUPPORTED_LOCALES="en_US.UTF-8"

ENV NODE_ENVI=production \
    AKAUNTING_VERSION=3.0.16 \
    COMPOSER_ALLOW_SUPERUSER=1

# install required dependencies
RUN apk --no-cache add git wget python3 nodejs yarn icu-dev zlib-dev libpng-dev libzip-dev

# install php extensions
RUN docker-php-ext-install bcmath intl gd zip 

# configure php extensions
# RUN docker-php-ext-configure opcache --enable-opcache

# setup working dir
WORKDIR /var/www/html

# install composer
COPY --from=composer:2.4 /usr/bin/composer /usr/bin/composer

# setup src
RUN git clone --branch $AKAUNTING_VERSION https://github.com/akaunting/akaunting.git . \
    && chown -R www-data:www-data /var/www/html \
    && chmod 755 /var/www/html

# copy env vars
COPY .env .

# run building
RUN composer prod 

# install yarn
## OPTIMIZATION STEPS
RUN rm -rf node_modules && mkdir node_modules && rm package-lock.json
RUN yarn install --frozen-lockfile && yarn cache clean 
RUN yarn run production

# php clear config
RUN php artisan key:generate \
    && php artisan config:cache

