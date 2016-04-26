FROM daocloud.io/php:7.0-fpm

COPY ./composer.phar /usr/sbin/composer
RUN chmod +x /usr/sbin/composer
RUN docker-php-ext-install pdo_mysql
