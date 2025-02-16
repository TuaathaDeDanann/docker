FROM debian:stable-slim

ENV PHP_VERSION 8.4.1
ENV NODE_VERSION 22.13.1

LABEL org.opencontainers.image.authors="NorAmarth<https://github.com/TuaathaDeDanann>"

ARG USER=app
ARG GROUP=app

ARG PHPIZE_DEPS="\
		autoconf \
		dpkg-dev dpkg \
		file \
		g++ \
		gcc \
		libc-dev \
		make \
		pkgconf \
		re2c"
ARG PHP_RUNTIME_DEPS="\
        curl \
        git \
        ca-certificates \
        libcurl4 \
        libonig5 \
        libedit2 \
        libsodium23 \
        libssl3 \
        libargon2-1 \
        libyaml-0-2 \
        libxml2 \
        libc-client2007e \
        librabbitmq4 \
        libpng16-16 \
        libjpeg62-turbo \
        libwebp7 \
        libfreetype6 \
        libpq5 \
        libzip4 \
        unzip \
        supervisor \
        nginx"
ARG APP_DIR=/app
ARG PHP_INI_DIR=/etc/php
ARG PHP_DOWNLOAD_DIR=/usr/local/src/tarballs
ARG PHP_SOURCE_DIR=/usr/local/src/php/source
ARG PHP_COMPILE_DIR=/usr/local/src/php/compile
ARG MODULES_DIR=/usr/local/src/modules
ARG MODULES_DOWNLOAD_DIR=$MODULES_DIR/tarballs
ARG PHP_ARCHIVE_FILE=php.tar.xz
ARG PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64"
ARG PHP_CPPFLAGS="$PHP_CFLAGS"
ARG PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"
ARG PHP_URL="https://www.php.net/get/php-$PHP_VERSION.tar.xz/from/this/mirror"
ARG PHP_COMPILE_DEPS="\
    libcurl4-openssl-dev \
    libedit-dev \
    libsodium-dev \
    libssl-dev \
    libxml2-dev \
    zlib1g-dev \
    libonig-dev \
    libxml2-dev \
    libargon2-0-dev \
    libc6-dev \
    libyaml-dev \
    libpng-dev \
    libwebp-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libgmp-dev \
    libzip-dev \
    libpq-dev \
    libc-client2007e-dev \
    libgssapi-krb5-2 \
    libkrb5-dev \
    librabbitmq-dev"
ARG PHP_CONFIG_OPTIONS="\
    --disable-all \
    --enable-option-checking=fatal \
    --enable-cli \
    --enable-fpm \
    --enable-bcmath \
    --enable-ctype \
    --enable-dom \
    --enable-fileinfo \
    --enable-ftp \
    --enable-gd \
    --enable-intl \
    --enable-filter \
    --enable-mbstring \
    --enable-mysqlnd \
    --enable-opcache \
    --enable-opcache-jit \
    --enable-pcntl \
    --enable-phar \
    --enable-pdo \
    --enable-posix \
    --enable-session \
    --enable-simplexml \
    --enable-sysvmsg \
    --enable-sysvsem \
    --enable-sysvshm \
    --enable-sockets \
    --enable-static \
    --enable-tokenizer \
    --enable-xml \
    --enable-xmlreader \
    --enable-xmlwriter \
    --with-curl \
    --with-freetype \
    --with-gmp \
    --with-iconv \
    --with-jpeg \
    --with-libedit \
    --with-libxml \
    --with-mhash \
    --with-openssl \
    --with-password-argon2 \
    --with-pcre-jit \
    --with-pgsql \
    --with-pdo-mysql \
    --with-pdo-pgsql \
    --with-sodium=shared \
    --with-zip \
    --with-zlib \
    --with-webp"

SHELL ["/bin/bash", "-c"]

RUN set -eux; \
	{ \
		echo 'Package: php*' \
		&& echo 'Pin: release *' \
		&& echo 'Pin-Priority: -1'; \
	} > /etc/apt/preferences.d/no-debian-php; \
	mkdir -p "$PHP_INI_DIR/conf.d" \
	&& mkdir -p $APP_DIR \
	&& groupadd -g 1000 -r $GROUP \
	&& useradd -u 1000 -r -g $GROUP $USER \
	&& mkdir -p /home/$USER \
	&& chown $USER:$GROUP /home/$USER \
	&& chown $USER:$GROUP $APP_DIR \
	&& chmod 777 $APP_DIR \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends \
	    $PHP_RUNTIME_DEPS \
	&& rm -rf /var/lib/apt/lists/*;
RUN set -eux; \
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
	    dirmngr \
	    autoconf \
	    automake \
	    autotools-dev \
	    libtool \
	    build-essential \
        xz-utils \
        $PHP_COMPILE_DEPS \
        $PHPIZE_DEPS; \
	rm -rf /var/lib/apt/lists/*; \
    mkdir -p $PHP_DOWNLOAD_DIR; \
    cd $PHP_DOWNLOAD_DIR; \
    curl -fsSL -o $PHP_ARCHIVE_FILE "$PHP_URL"; \
    export CFLAGS="$PHP_CFLAGS"; \
    export CPPFLAGS="$PHP_CPPFLAGS"; \
    export LDFLAGS="$PHP_LDFLAGS"; \
    gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
	debMultiarch="$(dpkg-architecture --query DEB_BUILD_MULTIARCH)"; \
	if [ ! -d /usr/include/curl ]; then \
		ln -sT "/usr/include/$debMultiarch/curl" /usr/local/include/curl; \
	fi; \
	mkdir -p $PHP_SOURCE_DIR; \
	tar -xf $PHP_DOWNLOAD_DIR/$PHP_ARCHIVE_FILE -C $PHP_SOURCE_DIR --strip-components=1; \
	rm -rf PHP_DOWNLOAD_DIR; \
	mkdir -p $PHP_COMPILE_DIR; \
    cd $PHP_SOURCE_DIR; \
    ./buildconf --force; \
    cd $PHP_COMPILE_DIR; \
    $PHP_SOURCE_DIR/configure \
        --build="$gnuArch" \
        --with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
        $PHP_CONFIG_OPTIONS;  \
    make -j "$(nproc)"; \
    find -type f -name '*.a' -delete; \
    make install; \
    find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; \
    make clean; \
    cp -v $PHP_SOURCE_DIR/php.ini-* "$PHP_INI_DIR/"; \
    cd /; \
    rm -rf $PHP_DOWNLOAD_DIR; \
    mkdir -p $MODULES_DOWNLOAD_DIR; \
    cd $MODULES_DOWNLOAD_DIR; \
    git clone -q https://github.com/igbinary/igbinary.git igbinary; \
    git clone -q --recurse-submodules https://github.com/phpredis/phpredis.git phpredis; \
    git clone -q https://github.com/krakjoe/apcu.git apcu; \
    git clone -q https://github.com/php/pecl-file_formats-yaml.git yaml; \
    cd $MODULES_DOWNLOAD_DIR/igbinary; \
    phpize; \
    ./configure CFLAGS="-O2 -g" \
        --enable-igbinary; \
    make -j "$(nproc)"; \
    find -type f -name '*.a' -delete; \
    make install; \
    cd $MODULES_DOWNLOAD_DIR/phpredis; \
    phpize; \
    ./configure \
        --enable-redis \
        --enable-redis-igbinary; \
    make -j "$(nproc)"; \
    find -type f -name '*.a' -delete; \
    make install; \
    cd $MODULES_DOWNLOAD_DIR/apcu; \
    phpize; \
    ./configure; \
    make -j "$(nproc)"; \
    find -type f -name '*.a' -delete; \
    make install; \
    cd $MODULES_DOWNLOAD_DIR/yaml; \
    phpize; \
    ./configure \
        --with-yaml ;\
    make -j "$(nproc)"; \
    find -type f -name '*.a' -delete; \
    make install; \
    cd / ;\
    rm -rf $PHP_SOURCE_DIR; \
    rm -rf $PHP_COMPILE_DIR; \
    rm -rf $MODULES_DIR; \
    echo "extension = apcu.so " > $PHP_INI_DIR/conf.d/10-apcu.ini; \
    echo "extension = yaml.so " > $PHP_INI_DIR/conf.d/10-yaml.ini; \
    echo "extension = sodium.so" > $PHP_INI_DIR/conf.d/10-sodium.ini; \
    echo "extension = igbinary.so" > $PHP_INI_DIR/conf.d/10-igbinary.ini; \
    echo "zend_extension = opcache.so" > $PHP_INI_DIR/conf.d/10-jit.ini; \
    echo "opcache.enable=1" >> $PHP_INI_DIR/conf.d/10-jit.ini; \
    echo "opcache.enable_cli=1" >> $PHP_INI_DIR/conf.d/10-jit.ini; \
    echo "opcache.jit_buffer_size=100M" >> $PHP_INI_DIR/conf.d/10-jit.ini; \
    echo "opcache.jit=1255" >> $PHP_INI_DIR/conf.d/10-jit.ini; \
    echo "extension = redis.so " > $PHP_INI_DIR/conf.d/20-redis.ini; \
    apt-mark auto '.*' > /dev/null; \
    [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
    find /usr/local -type f -executable -exec ldd '{}' ';' \
        | awk '/=>/ { print $(NF-1) }' \
        | sort -u \
        | xargs -r dpkg-query --search \
        | cut -d: -f1 \
        | sort -u \
        | xargs -r apt-mark manual; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /tmp/*; \
    mkdir -p /app/var/log/php \
    && mkdir -p /app/var/cache/ \
    && chown -R app:app /app \
    && chmod -R +x /app \
    && mkdir -p /var/log/php/ \
    && touch /var/log/php/access.log \
    && touch /var/log/php/global.log \
    && touch /var/log/php/error.log \
    && chown -R app:app /var/log/php; \
    && ARCH= && dpkgArch="$(dpkg --print-architecture)"
    && url -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
    && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
    && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    && node --version \
    && npm --version  

ADD config/php-fpm.conf /usr/local/etc/php-fpm.conf
ADD config/php.ini /etc/php/php.ini
ADD config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
ADD config/nginx.conf /etc/nginx/nginx.conf
ADD config/default.conf /etc/nginx/conf.d/nginx.conf
ADD config/fpm-pool.conf /etc/php/pools/pool.conf

STOPSIGNAL SIGQUIT

WORKDIR /app

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1/fpm-ping || exit 1
