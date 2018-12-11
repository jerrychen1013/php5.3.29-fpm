
FROM debian:jessie

# persistent / runtime deps
RUN set -xe \
	&& DEBIAN_FRONTEND=noninteractive apt-get update -qq \
	&& DEBIAN_FRONTEND=noninteractive apt-get install -qq -y --no-install-recommends --no-install-suggests \
		ca-certificates \
		curl \
		libpcre3 \
		librecode0 \
		libmysqlclient-dev \
		libsqlite3-0 \
		libxml2 \
# phpize deps
		autoconf \
		file \
		g++ \
		gcc \
		libc-dev \
		make \
		pkg-config \
		re2c \
		xz-utils \
# other tools
		sudo \
		cron \
		git \
		wget \
		python \
		vim \
		unzip \
		mysql-client \
		zip \
		apt-utils \
		supervisor \
		locales \
# php 5.3 needs older autoconf
# --enable-mysqlnd is included below because it's harder to compile after the fact the extensions are (since it's a plugin for several extensions, not an extension in itself)
		autoconf2.13 \ 
		libcurl4-openssl-dev \
		libpcre3-dev \
		libreadline6-dev \
		librecode-dev \
		libsqlite3-dev \
		libssl-dev \
		libxml2-dev \
		libfreetype6-dev \
		libjpeg62-turbo-dev \
                #====安裝mcrypt====
		libmcrypt-dev \
                #==================
		libpng-dev \
                libmagickwand-dev \
                libldb-dev \
                libldap2-dev \
                libsasl2-dev \
                libgeoip-dev \
                libpq-dev \
                libzip-dev \
                libbz2-dev \
                libgd2-dev \
                libjpeg-dev \
                libgif-dev \
	&& ln -s /usr/lib/x86_64-linux-gnu/libssl.so  /usr/lib \
	&& export PHP_OPENSSL_DIR=yes \
	&& DEBIAN_FRONTEND=noninteractive apt-get purge -qq -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
	&& rm -rf /var/lib/apt/lists/*

# install JP locale
RUN echo "ja_JP.UTF-8 UTF-8" >> /etc/locale.gen \
	locale-gen

ENV PHP_INI_DIR /usr/local/etc/php
RUN mkdir -p $PHP_INI_DIR/conf.d

# compile openssl, otherwise --with-openssl won't work
RUN set -xe \
	&& OPENSSL_VERSION="1.0.2g" \
	&& cd /tmp \
	&& mkdir openssl \
	&& curl -sL "https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz" -o openssl.tar.gz \
	# && curl -sL "https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz.asc" -o openssl.tar.gz.asc \
	&& tar -xzf openssl.tar.gz -C openssl --strip-components=1 \
	&& cd /tmp/openssl \
	&& ./config \
	&& make depend \
	&& make -j"$(nproc)" \
	&& make install \
	&& rm -rf /tmp/* \
	&& export PHP_OPENSSL_DIR=yes

ENV PHP_VERSION 5.3.29
COPY docker-php-* /usr/local/bin/
# COPY docker-php-source /usr/local/bin/


# php 5.3 needs older autoconf
# --enable-mysqlnd is included below because it's harder to compile after the fact the extensions are (since it's a plugin for several extensions, not an extension in itself)
RUN set -xe \
	&& mkdir -p /usr/src/php \
	&& curl -SL "http://php.net/get/php-$PHP_VERSION.tar.xz/from/this/mirror" -o /usr/src/php.tar.xz \
	&& curl -SL "http://php.net/get/php-$PHP_VERSION.tar.xz.asc/from/this/mirror" -o /usr/src/php.tar.xz.asc \
	&& cd /usr/src \
	&& docker-php-source extract \
	&& mkdir -p /usr/include/freetype2/freetype \
	&& ln -s /usr/include/freetype2/freetype.h /usr/include/freetype2/freetype/freetype.h \
	&& cd /usr/src/php \
	&& ./configure \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		--enable-fpm \
		--with-fpm-user=www-data \
		--with-fpm-group=www-data \
		--disable-cgi \
		--enable-mysqlnd \
		--with-mysql \
		--with-curl \
		--with-openssl=/usr/local/ssl \
		--with-readline \
		--with-recode \
		--with-zlib \
		--enable-ftp \
		--enable-mbstring \
		--enable-zip \
		--enable-pcntl \
		--with-pdo-mysql \
		--with-mysqli \
		--with-bz2 \
		--with-gd \
		--with-jpeg-dir \
		--with-up-dir \
		--with-pcre-regex \
# configure: error: Cannot find OpenSSL's libraries
		# --with-libdir="x86_64-linux-gnu" \
	&& make -j"$(nproc)" \
	&& make install \
	&& make clean \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \ 
	# find extensions's so and put it in php extensions folder & "produce ini files" in php/conf.d
	&& docker-php-ext-install -j$(nproc) exif hash sockets xml zip pcntl bcmath gettext\
	&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/  \
	&& docker-php-ext-install -j$(nproc) gd \
	&& docker-php-ext-install -j$(nproc) pdo_mysql mysqli \
	# && docker-php-ext-install -j$(nproc) pgsql pdo_pgsql ctype\
	&& cd / \
	&& docker-php-source delete \
	&& DEBIAN_FRONTEND=noninteractive apt-get purge -qq -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
	&& rm -rf /var/lib/apt/lists/*

#安裝composer
RUN EXPECTED_SIGNATURE=$(wget -q -O - https://composer.github.io/installer.sig); \
	php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"; \
	ACTUAL_SIGNATURE=$(php -r "echo hash_file('SHA384', 'composer-setup.php');"); \
	php composer-setup.php; \
	php -r "unlink('composer-setup.php');"; \
	mv composer.phar /usr/local/bin/composer; 

RUN  pecl install xdebug-2.2.4 \
    #  pecl install redis \
    #  pecl install imagick \
     && docker-php-ext-enable xdebug

WORKDIR /var/www/html
# RUN mkdir -p /usr/local/etc/php-fpm.d
COPY php-fpm.conf /usr/local/etc/
RUN	echo 'export TERM=xterm-256color' >> ~/.bashrc; \
	echo 'alias ll="ls -al"' >> ~/.bashrc; \
	mkdir -p /var/lib/php/session; \
	chown -R www-data:www-data /var/lib/php/session 

EXPOSE 9000
CMD ["php-fpm"]
