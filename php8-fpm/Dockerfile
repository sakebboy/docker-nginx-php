FROM php:8.3.12-fpm-bookworm

ARG NODE_VERSION=18

ENV ACCEPT_EULA=Y

# 與上游 php-sapnwrfc 相容之 PHP 8 原始碼版本（倉庫名仍為 php7-sapnwrfc）
ARG SAPNWRFC_VERSION=2.1.0


RUN apt-get update && apt-get install -y --no-install-recommends iputils-ping \
    && rm -rf /var/lib/apt/lists/*

#RUN pecl install xdebug && docker-php-ext-enable xdebug

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        apt-utils \
        libxml2-dev \
        gnupg \
        apt-transport-https \
        curl \
        libmemcached-dev \
        zlib1g-dev \
        libpq-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libfreetype6-dev \
        libssl-dev \
        libmcrypt-dev \
        zip \
        unzip \
        build-essential \
        libaio1 \
        libzip-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* \
    && rm -f /var/log/lastlog /var/log/faillog \
    && pecl install mcrypt-1.0.7

RUN apt-get update \
    && apt-get install -y --no-install-recommends htop \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# PHP 8.3 需 Swoole 5.x
RUN pecl install swoole-5.1.4

RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

RUN npm install apidoc@0.17.6 -g

RUN apt-get update \
    && apt-get install -y --no-install-recommends vim \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
    libltdl-dev \
    libtool \
    autoconf \
    automake \
    make \
    g++ \
    unixodbc \
    unixodbc-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Microsoft ODBC Driver 18 for SQL Server（Debian 12 / bookworm）
RUN curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg \
    && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-prod.gpg] https://packages.microsoft.com/debian/12/prod bookworm main" > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \
    && ACCEPT_EULA=Y apt-get install -y --no-install-recommends msodbcsql18 mssql-tools18 unixodbc-dev \
    && rm -rf /var/lib/apt/lists/*

ENV PATH="${PATH}:/opt/mssql-tools18/bin"

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/bin/install-php-extensions

RUN apt-get update \
    && apt-get install -y --no-install-recommends unixodbc odbcinst \
    && rm -rf /var/lib/apt/lists/*

#composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install required extensions（redis 改由相容 PHP 8 之版本安裝，避免舊 pecl 4.0.1）
RUN chmod uga+x /usr/bin/install-php-extensions \
    && sync
RUN install-php-extensions pcntl
RUN install-php-extensions pdo_sqlsrv
RUN install-php-extensions redis
RUN install-php-extensions sqlsrv
RUN install-php-extensions mbstring
RUN install-php-extensions ldap
RUN install-php-extensions ssh2
RUN docker-php-ext-enable swoole

RUN docker-php-ext-enable mcrypt
RUN apt-get update \
    && apt-get install -y --no-install-recommends libicu-dev \
    && docker-php-ext-install intl \
    && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-install mysqli
RUN docker-php-ext-install pdo
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-install bcmath
RUN docker-php-ext-install soap
RUN docker-php-ext-install zip
RUN docker-php-ext-install opcache
RUN docker-php-ext-install ftp
RUN docker-php-ext-configure pdo_odbc --with-pdo-odbc=unixODBC,/usr \
    && docker-php-ext-install pdo_odbc
RUN set -eux; \
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-install gd

# https://github.com/docker-library/php/issues/103#issuecomment-160772802
RUN set -ex \
    && cd /usr/src/ && tar -xf php.tar.xz && mv php-8* php \
    && cd /usr/src/php/ext/odbc \
    && phpize \
    && sed -ri 's@^ *test +"\$PHP_.*" *= *"no" *&& *PHP_.*=yes *$@#&@g' configure \
    && ./configure --with-unixODBC=shared,/usr \
    && docker-php-ext-install odbc


# ORACLE oci
RUN mkdir -p /opt/oracle \
    && cd /opt/oracle

ADD php-fpm/instantclient-basic-linux.x64-12.1.0.2.0.zip /opt/oracle
ADD php-fpm/instantclient-sdk-linux.x64-12.1.0.2.0.zip /opt/oracle

RUN unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
    && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
    && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
    && rm -rf /opt/oracle/*.zip

ENV LD_LIBRARY_PATH=/opt/oracle/instantclient_12_1

# PHP 8 使用 OCI8 3.x
RUN echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-3.4.0 \
    && docker-php-ext-enable oci8 \
    && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_12_1,12.1 \
    && docker-php-ext-install pdo_oci


# Install SAP Netweaver RFC SDK extensions
ADD php8-fpm/sap/nwrfc750P_11-70002752/nwrfcsdk /usr/sap/nwrfcsdk
WORKDIR /usr/sap
RUN echo /usr/sap/nwrfcsdk/lib > /etc/ld.so.conf.d/nwrfcsdk.conf \
    && ldconfig \
    && curl --fail --location --remote-name "https://github.com/gkralik/php7-sapnwrfc/archive/${SAPNWRFC_VERSION}.tar.gz" \
    && tar xzf "${SAPNWRFC_VERSION}.tar.gz"
WORKDIR /usr/sap/php7-sapnwrfc-${SAPNWRFC_VERSION}
RUN phpize \
    && ./configure --with-sapnwrfc=/usr/sap/nwrfcsdk \
    && make \
    && make install

#RUN docker-php-ext-enable sapnwrfc

WORKDIR /var/www
