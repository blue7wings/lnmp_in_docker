FROM daocloud.io/library/ubuntu
COPY ./sources.list /etc/apt/sources.list
RUN apt-get update
RUN apt-get -y install php5
RUN apt-get -y install git
COPY ./composer.phar /usr/local/bin/composer
RUN chmod +x /usr/local/bin/composer
RUN composer config -g repo.packagist composer https://packagist.phpcomposer.com
