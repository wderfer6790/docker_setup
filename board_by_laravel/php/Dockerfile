FROM php:8.4-fpm-alpine

RUN apk add --no-cache \
    bash \
    git \
    unzip \
    icu-dev \
    libzip-dev \
    mysql-client \
    && docker-php-ext-install -j$(nproc) \
    pdo_mysql \
    mysqli \
    intl \
    zip

RUN git config --system --add safe.directory /var/git/laravel.wderfer6790.kr    

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

WORKDIR /var/git