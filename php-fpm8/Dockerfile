FROM php:8.3-fpm-bookworm

# === 基本設定 ===
ARG NODE_VERSION=16
ENV ACCEPT_EULA=Y
ENV DEBIAN_FRONTEND=noninteractive
ENV LD_LIBRARY_PATH=/opt/oracle/instantclient_12_1:${LD_LIBRARY_PATH}

# === 系統套件 ===
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        curl unzip gnupg vim \
        libmemcached-dev libz-dev libpq-dev \
        libjpeg-dev libpng-dev libfreetype6-dev libssl-dev \
        libwebp-dev libxpm-dev libonig-dev \
        unixodbc unixodbc-dev odbcinst; \
    rm -rf /var/lib/apt/lists/*

# === PHP 擴充套件 ===
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        libjpeg62-turbo-dev libpng-dev libfreetype6-dev libwebp-dev \
        libxml2-dev libicu-dev libzip-dev \
        libonig-dev libpq-dev zlib1g-dev; \
    docker-php-ext-configure gd \
        --with-jpeg \
        --with-webp \
        --with-freetype; \
    docker-php-ext-install -j$(nproc) \
        gd pdo_mysql pdo_pgsql bcmath soap zip opcache intl mysqli; \
    rm -rf /var/lib/apt/lists/*
	
	
# === 系統相依套件 ===
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    libzip-dev \
    libpng-dev \
    libxml2-dev \
    libssl-dev \
    libssh2-1-dev \
    pkg-config \
    && docker-php-ext-install zip pdo_mysql

# === PECL 套件安裝 ===
RUN pecl install \
        swoole-5.1.3 \
        redis-6.0.2 \
        ssh2-1.4.1 \
        xdebug-3.3.0 \
    && docker-php-ext-enable swoole redis ssh2 xdebug
	
	
# === Node.js & apidoc ===
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g apidoc@0.17.6

# === Composer ===
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# === Microsoft SQL Server 驅動 ===
RUN set -eux; \
    apt-get update && apt-get install -y curl gnupg2 ca-certificates; \
    mkdir -p /etc/apt/keyrings; \
    curl -sSL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o /etc/apt/keyrings/microsoft.gpg; \
    chmod 644 /etc/apt/keyrings/microsoft.gpg; \
    echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/microsoft.gpg] https://packages.microsoft.com/debian/12/prod bookworm main" > /etc/apt/sources.list.d/mssql-release.list; \
    apt-get update; \
    ACCEPT_EULA=Y apt-get install -y msodbcsql17 mssql-tools unixodbc-dev; \
    pecl install sqlsrv pdo_sqlsrv; \
    docker-php-ext-enable sqlsrv pdo_sqlsrv	
	
# === Oracle Instant Client ===
# 將 zip 檔案複製鏡映像
COPY oracle/instantclient-basic-linux.x64-19.22.0.0.0dbru.zip /opt/oracle/
COPY oracle/instantclient-sdk-linux.x64-19.22.0.0.0dbru.zip /opt/oracle/

# unzip安裝oracle
RUN set -eux; \
    apt-get update && apt-get install -y libaio1 unzip; \
    cd /opt/oracle; \
    unzip -o instantclient-basic-linux.x64-19.22.0.0.0dbru.zip; \
    rm -rf META-INF; \
    unzip -o instantclient-sdk-linux.x64-19.22.0.0.0dbru.zip; \
    rm -f instantclient-*.zip; \
    echo /opt/oracle/instantclient_19_22 > /etc/ld.so.conf.d/oracle-instantclient.conf; \
    ldconfig; \
    echo 'instantclient,/opt/oracle/instantclient_19_22/' | pecl install oci8-3.3.0; \
    docker-php-ext-enable oci8; \
    docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_19_22,19.22; \
    docker-php-ext-install pdo_oci

	
# === Xdebug 設定 ===
RUN { \
        echo "zend_extension=xdebug.so"; \
        echo "xdebug.mode=debug,develop,coverage"; \
        echo "xdebug.start_with_request=yes"; \
        echo "xdebug.discover_client_host=true"; \
        echo "xdebug.client_port=9003"; \
        echo "xdebug.log=/tmp/xdebug.log"; \
    } > /usr/local/etc/php/conf.d/xdebug.ini

# 可透過環境變數控制 (ex: docker run -e XDEBUG_MODE=off ...)
ENV XDEBUG_MODE=debug,develop,coverage

WORKDIR /var/www
