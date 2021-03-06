FROM php:7.3-fpm as scratch

ARG APP_ENV
ENV APP_ENV=${APP_ENV} \
    PHP_OPCACHE_VALIDATE_TIMESTAMPS="0" \
    PHP_OPCACHE_MAX_ACCELERATED_FILES="10000" \
    PHP_OPCACHE_MEMORY_CONSUMPTION="192" \
    PHP_OPCACHE_MAX_WASTED_PERCENTAGE="10"

# Add php config
COPY .docker/php-fpm/php.ini /usr/local/etc/php/php.ini
COPY .docker/php-fpm/www.conf /usr/local/etc/php-fpm.d/www.conf

RUN apt-get update && apt-get install -y libxml2-dev libpq-dev libicu-dev unzip wget git \
    && docker-php-ext-install pdo pdo_mysql intl tokenizer xml pcntl mbstring bcmath opcache

RUN apt-get install -y libz-dev zlibc libmemcached-dev \
    && pecl channel-update pecl.php.net \
    && pecl install -o -f apcu \
    && docker-php-ext-enable apcu \
    && pecl clear-cache

RUN wget https://getcomposer.org/installer -O - -q | php -- --install-dir=/bin --filename=composer --quiet

ENV COMPOSER_ALLOW_SUPERUSER 1

WORKDIR /var/www/html

# DEV env (local)
FROM scratch as dev

RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

# PROD env (kubernetes)
FROM scratch as prod

RUN /usr/bin/composer install --no-ansi --no-dev -n --no-plugins --no-progress --no-scripts --no-suggest -o \
    && chown -R www-data:www-data .

USER www-data:www-data
CMD ["php-fpm"]
EXPOSE 9000