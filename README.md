# docker_apache_php
Building a customized Docker Image - Apache and PHP - Supported in Openstack also(nova-docker)

First Create a following Directory Structure:

./Dockerfile
./apache-config.conf
./www/index.php
Dockerfile contains should be as follows:

FROM ubuntu:latest
MAINTAINER Murali muralidharans@infinite.com

# Install apache, PHP, and supplimentary programs. openssh-server, curl, and lynx-cur are for debugging the container.
RUN apt-get update && apt-get -y upgrade && DEBIAN_FRONTEND=noninteractive apt-get -y install \
    apache2 php7.0 php7.0-mysql libapache2-mod-php7.0 curl lynx-cur

# Enable apache mods.
RUN a2enmod php7.0
RUN a2enmod rewrite

# Update the PHP.ini file, enable <? ?> tags and quieten logging.
RUN sed -i "s/short_open_tag = Off/short_open_tag = On/" /etc/php/7.0/apache2/php.ini
RUN sed -i "s/error_reporting = .*$/error_reporting = E_ERROR | E_WARNING | E_PARSE/" /etc/php/7.0/apache2/php.ini

# Manually set up the apache environment variables
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

# Expose apache.
EXPOSE 80

# Copy this repo into place.
ADD www /var/www/site

# Update the default apache site with the config we created.
ADD apache-config.conf /etc/apache2/sites-enabled/000-default.conf

# By default start up apache in the foreground, override with /bin/bash for interative.
CMD /usr/sbin/apache2ctl -D FOREGROUND




apache-config.conf:

<VirtualHost *:80>
  ServerAdmin me@mydomain.com
  DocumentRoot /var/www/site

  <Directory /var/www/site/>
      Options Indexes FollowSymLinks MultiViews
      AllowOverride All
      Order deny,allow
      Allow from all
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>



www/index.php:


<? echo "<p>Hello! Test Working fine?</p>"; ?>


Docker Image Build:

 docker build -t muraliselva10/apache_php .


Verifying the working of Docker Image:

docker run -p 8080:80 -d muraliselva10/apache_php

docker run -i -t -p 8080:80 muraliselva10/apache_php /bin/bash
apachectl start
Push to Docker Hub:
docker push muraliselva10/apache_php
In Openstack:
docker pull muraliselva10/apache_php
docker save muraliselva10/apache_php | openstack image create muraliselva10/apache_php --public --container-format docker --disk-format raw
