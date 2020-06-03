FROM debian:buster

MAINTAINER m.krzaczkowski@yetiforce.com

ARG DEBIAN_FRONTEND=noninteractive
ARG DB_ROOT_PASS=1r2VdePVnNxluabdGuqh

ENV PHP_VER 7.3
ENV DB_USER_NAME yetiforce
ENV DB_USER_PASS Q4WK2yRUpliyjMRivDJE
ENV DB_PORT 3306
#INSTALL_MODE = PROD , DEV
ENV INSTALL_MODE PROD

ENV PROVIDER docker.hub.official

RUN apt-get update && apt-get install -y --no-install-recommends apt-utils curl openssl wget ca-certificates apt-transport-https lsb-release gnupg && apt-get -y autoclean

RUN wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
RUN echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list
RUN	curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

RUN apt-get update
RUN apt-get install -y --no-install-recommends mariadb-server mariadb-client nginx nginx-extras "php${PHP_VER}"-fpm "php${PHP_VER}"-mysql "php${PHP_VER}"-curl "php${PHP_VER}"-intl "php${PHP_VER}"-gd "php${PHP_VER}"-fpm "php${PHP_VER}"-bcmath "php${PHP_VER}"-soap "php${PHP_VER}"-ldap "php${PHP_VER}"-imap "php${PHP_VER}"-xml "php${PHP_VER}"-cli "php${PHP_VER}"-zip "php${PHP_VER}"-json "php${PHP_VER}"-opcache "php${PHP_VER}"-mbstring php-apcu php-imagick php-sodium zip unzip mc htop openssh-server cron && apt-get -y autoclean

RUN dpkg --get-selections | grep php

RUN rm /var/www/html/index.nginx-debian.html
RUN wget https://download.yetiforce.com/developer-for-docker.zip -P /var/www/html/ &> /dev/null
RUN unzip /var/www/html/developer-for-docker.zip -d /var/www/html/ > /dev/null
RUN rm /var/www/html/developer-for-docker.zip

RUN cp /var/www/html/tests/setup/db/mysql.cnf /etc/mysql/mariadb.conf.d/50-server.cnf
RUN cp /var/www/html/tests/setup/nginx/vhost.conf /etc/nginx/sites-available/default
RUN cp /var/www/html/tests/setup/crons.conf /etc/cron.d/yetiforcecrm
RUN cp /var/www/html/tests/setup/php/prod.ini /etc/php/7.3/mods-available/yetiforce.ini
RUN cp /var/www/html/tests/setup/docker_entrypoint.sh /
RUN rm /var/www/html/.user.ini
RUN rm /var/www/html/public_html/.user.ini

RUN	service mysql start; \
	mysql -uroot mysql; \
	mysqladmin password "$DB_ROOT_PASS"; \
	echo "UPDATE mysql.user SET Password=PASSWORD('$DB_ROOT_PASS') WHERE User='root';" | mysql --user=root;\
	echo "DELETE FROM mysql.user WHERE User='';" | mysql --user=root;\
	echo "DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');" | mysql --user=root; \
	echo "DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%';" | mysql --user=root; \
	echo "CREATE DATABASE yetiforce;" | mysql --user=root;\
	echo "CREATE USER 'yetiforce'@'localhost' IDENTIFIED BY '$DB_USER_PASS';" | mysql --user=root;\
	echo "GRANT ALL PRIVILEGES ON yetiforce.* TO 'yetiforce'@'localhost';" | mysql --user=root;\
	echo "FLUSH PRIVILEGES;" | mysql --user=root

RUN crontab /etc/cron.d/yetiforcecrm
RUN ln -s /etc/php/7.3/mods-available/yetiforce.ini /etc/php/7.3/cli/conf.d/30-yetiforce.ini
RUN ln -s /etc/php/7.3/mods-available/yetiforce.ini /etc/php/7.3/fpm/conf.d/30-yetiforce.ini
RUN	chmod +x /docker_entrypoint.sh
RUN php -f /var/www/html/tests/setup/docker_post_install.php
RUN rm -rf /var/www/html/tests/
RUN chown -R www-data:www-data /var/www/
RUN echo "PROVIDER=docker.hub.official" > /etc/environment

WORKDIR /var/www/html

EXPOSE 80
EXPOSE 3306

ENTRYPOINT [ "/docker_entrypoint.sh" ]
