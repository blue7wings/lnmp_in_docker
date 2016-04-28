FROM daocloud.io/php:7.0-fpm

COPY ./composer.phar /usr/sbin/composer
COPY ./php.ini /usr/local/etc/php/
RUN apt-get update
RUN chmod +x /usr/sbin/composer
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-install mbstring
RUN apt-get install -y libpcre3 libpcre3-dev \
    && pecl install oauth \
    && touch /usr/local/etc/php/conf.d/oauth.ini \
    && echo 'extension=oauth.so' >> /usr/local/etc/php/conf.d/oauth.ini
