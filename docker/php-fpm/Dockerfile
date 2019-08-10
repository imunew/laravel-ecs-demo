FROM php:7-fpm

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y git zlib1g-dev zip unzip libzip-dev
RUN docker-php-ext-install zip

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer
RUN composer global require hirak/prestissimo

RUN mkdir -p /app
ADD . /app/

WORKDIR /app

RUN composer install
RUN cp .env.example .env
RUN php artisan key:generate
RUN chmod -R a+w storage/ bootstrap/cache
