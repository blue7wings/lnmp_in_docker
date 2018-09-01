FROM php:7.0-fpm

COPY ./php.ini /usr/local/etc/php/

RUN docker-php-ext-install mysqli
RUN docker-php-ext-install mbstring
RUN apt-get update \
    && apt-get install -y libpcre3 libpcre3-dev \
    && pecl install oauth \
    && touch /usr/local/etc/php/conf.d/oauth.ini \
    && echo 'extension=oauth.so' >> /usr/local/etc/php/conf.d/oauth.ini