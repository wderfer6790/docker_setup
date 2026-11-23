FROM php:8.4.27-fpm-alpine

RUN apk add --no-cache \
    bash \
    git \
    unzip \
    tzdata \
    icu-dev \
    libzip-dev \
    mysql-client \
    && docker-php-ext-install -j$(nproc) \
    pdo_mysql \
    mysqli \
    intl \
    zip

RUN --mount=type=bind,from=ghcr.io/php/pie:bin,source=/pie,target=/usr/local/bin/pie \
    apk add --no-cache --virtual .pie-build-deps $PHPIZE_DEPS automake libtool \
    && pie install --no-cache phpredis/phpredis \
    && apk del .pie-build-deps \
    && php --ri redis

RUN git config --system --add safe.directory /var/git/laravel.wderfer6790.kr    

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer
COPY conf.d/99-custom.ini /usr/local/etc/php/conf.d/99-custom.ini

WORKDIR /var/git