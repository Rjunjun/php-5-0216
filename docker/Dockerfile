FROM php:5.5-apache

# Install PHP extension dependencies (if any)
#RUN apt-get update \
#    && apt-get -y --no-install-recommends install <package_name>

# Copy PHP configuration
# Edit docker/php.ini and customize it to your needs
COPY docker/php.ini /usr/local/etc/php/php.ini

# Copy custom command docker-php-pecl-install
COPY docker/docker-php-pecl-install /usr/local/bin/

# Install PHP extensions
RUN docker-php-ext-install mysql
RUN docker-php-ext-install mysqli
RUN docker-php-ext-install pdo
RUN docker-php-ext-install pdo_mysql
#ioncube 5.5
RUN cd /tmp \
    && curl -o ioncube.tar.gz http://downloads3.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz \
    && tar -xvvzf ioncube.tar.gz \
    && mv ioncube/ioncube_loader_lin_5.5.so /usr/local/lib/php/extensions/* \
    && rm -Rf ioncube.tar.gz ioncube \
    && echo "zend_extension=ioncube_loader_lin_5.5.so" > /usr/local/etc/php/conf.d/00_docker-php-ext-ioncube_loader_lin_5.5.ini
#init index.php
RUN cd /var/www/html/ \
    && echo "<?php phpinfo(); ?>" > index.php

#google drive
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    rm -rf /var/lib/apt/lists/*

# Install PECL extensions
#RUN docker-php-pecl-install <package_name-version> (versions available in PECL repository allowed)
# global environment settings
ENV PLATFORM_ARCH="amd64"
ARG RCLONE_VERSION="current"

# s6 environment settings
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2
ENV S6_KEEP_ENV=1

# install packages
RUN \
 apk update && \
 apk add --no-cache \
 ca-certificates

# install build packages
RUN \
 apk add --no-cache --virtual=build-dependencies \
		wget \
		curl \
		unzip && \
# add s6 overlay
 OVERLAY_VERSION=$(curl -sX GET "https://api.github.com/repos/just-containers/s6-overlay/releases/latest" \
	| awk '/tag_name/{print $4;exit}' FS='[""]') && \
 curl -o \
	/tmp/s6-overlay.tar.gz -L \
	"https://github.com/just-containers/s6-overlay/releases/download/${OVERLAY_VERSION}/s6-overlay-${PLATFORM_ARCH}.tar.gz" && \
 tar xfz \
	/tmp/s6-overlay.tar.gz -C / && \
 cd tmp && \
 wget -q https://downloads.rclone.org/v${RCLONE_VERSION}/rclone-v${RCLONE_VERSION}-linux-${PLATFORM_ARCH}.zip && \
 unzip /tmp/rclone-v${RCLONE_VERSION}-linux-${PLATFORM_ARCH}.zip && \
 mv /tmp/rclone-*-linux-${PLATFORM_ARCH}/rclone /usr/bin && \
 apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community \
	shadow && \
# cleanup
 apk del --purge \
	build-dependencies && \
 rm -rf \
	/tmp/* \
	/var/tmp/* \
	/var/cache/apk/*

# create abc user
RUN \
	groupmod -g 1000 users && \
	useradd -u 911 -U -d /config -s /bin/false abc && \
	usermod -G users abc && \
# create some files / folders
	mkdir -p /config /app /defaults /data && \
	touch /var/lock/rclone.lock

# add local files
COPY root/ /

VOLUME ["/config"]

ENTRYPOINT ["/init"]
# If you are using XDebug, this might be useful, if not, ignore it
# You additionally will have to customize docker/xdebug-ini-overrides.ini file to your needs
#RUN docker-php-pecl-install xdebug-2.5.4
#COPY docker/xdebug-ini-overrides.ini /usr/local/etc/php/conf.d/docker-php-pecl-xdebug.ini

# Configure Apache VHOSTS
#COPY docker/mysite.com.conf /etc/apache2/sites-available/mysite.com.conf
#RUN a2ensite <mysite.com.conf>

# Activate Apache modules
#RUN a2enmod <module_name>
#RUN service apache2 restart

# Set the workdir
WORKDIR /var/www/docker-lamp-php-5
