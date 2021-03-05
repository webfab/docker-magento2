FROM php:7.4-fpm

# Container containing php-fpm and php-cli to run and interact with eZ Platform and other Symfony projects
#
# It has two modes of operation:
# - (run.sh cmd) [default] Reconfigure eZ Platform/Publish based on provided env variables and start php-fpm
# - (bash|php|composer) Allows to execute composer, php or bash against the image

# Set defaults for variables used by run.sh
ENV COMPOSER_HOME=/root/.composer

# Get packages that we need in container
RUN apt-get update -q -y \
    && apt-get install -q -y apt-transport-https --no-install-recommends \
        ca-certificates \
        curl \
        acl \
        sudo \
# Needed for the php extensions we enable below
        libfreetype6 \
        libxpm4 \
        libmemcachedutil2 \
	libjpeg-dev \
	libmagickwand-dev \
	libpng-dev \
	libzip-dev \
	libxslt-dev \
	libcurl4-openssl-dev \
# Need For xhprof UI
	graphviz \
# git & unzip needed for composer, unless we document to use dev image for composer install
# unzip needed due to https://github.com/composer/composer/issues/4471
        unzip \
        git \
        && docker-php-ext-configure gd \ 
        && docker-php-ext-configure intl \
        && docker-php-ext-configure pdo_mysql --with-pdo-mysql=mysqlnd \
        && docker-php-ext-install -j$(nproc) gd pdo_mysql intl\
        && rm -rf /var/lib/apt/lists/*

RUN apt-get dist-upgrade
# Install and configure php plugins
RUN set -ex; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libmagickwand-dev \
		libpng-dev \
		libmemcached-dev \
		libmemcached11 \
		libmemcached-dev \
		zlib1g-dev \
        libwebp-dev \
		libjpeg62-turbo-dev \
        	libmcrypt-dev \
        	libpng-dev \
        	libxml2-dev \
        	graphviz \
		libonig-dev \
	; \
	\
	docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp; \
	docker-php-ext-install -j "$(nproc)" \
		bcmath \
		exif \
		gd \
		mysqli \
		zip \
        pdo \
		dom \
		xml \
		soap \
		curl \
        xsl\
        sockets \
	; \
	pecl install imagick-3.4.4 memcached-3.1.3; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*
# Install redis (manualy build in order to be able to enable igbinary)
# Set timezone
RUN echo "UTC" > /etc/timezone && dpkg-reconfigure --frontend noninteractive tzdata

# set recommended PHP.ini settings
RUN set -ex \
	&& cd /usr/local/etc \
	&& if [ -d php-fpm.d ]; then \
		# for some reason, upstream's php-fpm.conf.default has "include=NONE/etc/php-fpm.d/*.conf"
		sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
		cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
	else \
		# PHP 5.x doesn't use "include=" by default, so we'll create our own simple config that mimics PHP 7+ for consistency
		mkdir php-fpm.d; \ 
		cp php-fpm.conf.default php-fpm.d/www.conf; \
		{ \
			echo '[global]'; \
			echo 'include=etc/php-fpm.d/*.conf'; \
		} | tee php-fpm.conf; \
	fi \
	&& { \
		echo '[global]'; \
		echo 'error_log = /proc/self/fd/2'; \
		echo; \
		echo '[www]'; \
		echo '; if we send this to /proc/self/fd/1, it never appears'; \
		echo 'access.log = /proc/self/fd/2'; \
		echo; \
		echo 'clear_env = no'; \
		echo; \
		echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
		echo 'catch_workers_output = yes'; \
	} | tee php-fpm.d/docker.conf \
	&& { \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = [::]:9000'; \
	} | tee php-fpm.d/zz-docker.conf

# Set pid file to be able to restart php-fpm
RUN sed -i "s@^\[global\]@\[global\]\n\npid = /run/php-fpm.pid@" ${PHP_INI_DIR}-fpm.conf

# COPY conf.d/blackfire.ini ${PHP_INI_DIR}/conf.d/blackfire.ini

# Xhprof Fork PHP7 Install
# RUN cd /opt/ && git clone "https://github.com/tideways/php-xhprof-extension.git" && cd php-xhprof-extension && phpize && ./configure && make && sudo make install

# Install Xhprof UI

#RUN cd /var/www && git clone "https://github.com/preinheimer/xhprof.git"

# Create Composer directory (cache and auth files) & Get Composer
RUN mkdir -p $COMPOSER_HOME \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# As application is put in as volume we do all needed operation on run
COPY scripts /scripts

# RUN docker-php-ext-install xsl

# Add some custom config
COPY conf.d/php.ini ${PHP_INI_DIR}/conf.d/php.ini

# PHP extention 
RUN docker-php-ext-install bcmath

RUN chmod 755 /scripts/*.sh

# Needed for docker-machine
RUN usermod -u 1000 www-data

WORKDIR /var/www

ENTRYPOINT ["/scripts/docker-entrypoint.sh"]

CMD php-fpm

EXPOSE 9000
