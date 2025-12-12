FROM php:7.4-fpm-bullseye

ARG NODE_VERSION=18

ENV ACCEPT_EULA=Y

ARG SAPNWRFC_VERSION=1.4.0


RUN apt-get update && apt-get install iputils-ping -y
#RUN pecl install xdebug && docker-php-ext-enable xdebug

# Update OS packages and install required ones.
# Install selected extensions and other stuff
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
		apt-utils \
		libxml2-dev \
		gnupg \
		apt-transport-https \
		curl\
		libmemcached-dev \
		libz-dev \
		libpq-dev \
		libjpeg-dev \
		libpng-dev \
		libfreetype6-dev \
		libssl-dev \
		libmcrypt-dev \
		zip \
		unzip \
		build-essential \
		libaio1 \
		libzip-dev \
		apt-transport-https \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* \
	&& rm /var/log/lastlog /var/log/faillog \
	&& pecl install redis-4.0.1 \
	&& pecl install mcrypt-1.0.4

RUN apt-get update \
    && apt-get install -y htop \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN pecl install swoole-4.8.13
	
RUN curl -sL https://deb.nodesource.com/setup_$NODE_VERSION.x | bash 
RUN apt-get install -y nodejs 
# RUN apt-get install npm
	
RUN apt-get update \
	&& pecl install mcrypt

RUN npm install apidoc@0.17.6 -g

RUN apt-get update \
	&& apt-get install -y vim

RUN rm -rf /var/lib/apt/lists/*

# 必要套件
RUN apt-get update && apt-get install -y \
    libltdl-dev \
    libtool \
    autoconf \
    automake \
    make \
    g++ \
    unixodbc \
    unixodbc-dev \
    && apt-get clean
	
# Microsoft ODBC Driver 17 for SQL Server（Debian 11 repo）
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/11/prod.list \
       > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \
    && ACCEPT_EULA=Y apt-get install -y msodbcsql17 mssql-tools


RUN apt-get update && apt-get install -y \
    msodbcsql17 \	
    mssql-tools \	
    unixodbc-dev

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/bin/install-php-extensions

# Install Dependencies
RUN apt-get install -y unixodbc
RUN apt-get install -y unixodbc-dev
#RUN apt-get install -y libgss3
RUN apt-get install -y odbcinst
#RUN apt-get install -y locales
#RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen

#composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
	
# Install required extensions
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
RUN docker-php-ext-install intl
RUN docker-php-ext-install mysqli
RUN docker-php-ext-install pdo
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-install bcmath
RUN docker-php-ext-install soap
RUN docker-php-ext-install zip
RUN docker-php-ext-install opcache
RUN docker-php-ext-configure pdo_odbc --with-pdo-odbc=unixODBC,/usr \
 && docker-php-ext-install pdo_odbc
RUN set -eux; \
	docker-php-ext-configure gd \
            --prefix=/usr \
            --with-jpeg \
            --with-freetype; \
    docker-php-ext-install gd

#RUN rm -rf /var/lib/apt/lists/*

# cannot install odbc wihtout the below because of https://github.com/docker-library/php/issues/103#issuecomment-160772802
RUN set -x \
    && cd /usr/src/ && tar -xf php.tar.xz && mv php-7* php \
    && cd /usr/src/php/ext/odbc \
    && phpize \
    && sed -ri 's@^ *test +"\$PHP_.*" *= *"no" *&& *PHP_.*=yes *$@#&@g' configure \
    && ./configure --with-unixODBC=shared,/usr \ 
    && docker-php-ext-install odbc


# ORACLE oci 
RUN mkdir /opt/oracle \
    && cd /opt/oracle    
	
ADD instantclient-basic-linux.x64-12.1.0.2.0.zip /opt/oracle
ADD instantclient-sdk-linux.x64-12.1.0.2.0.zip /opt/oracle

# Install Oracle Instantclient
RUN  unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
    && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
    && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
    && rm -rf /opt/oracle/*.zip
    
ENV LD_LIBRARY_PATH  /opt/oracle/instantclient_12_1:${LD_LIBRARY_PATH}
    
# Install Oracle extensions
RUN echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-2.2.0 \ 
      && docker-php-ext-enable \
               oci8 \ 
       && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_12_1,12.1 \
       && docker-php-ext-install \
               pdo_oci 


# Install SAP Netweaver RFC SDK extensions
ADD sap/nwrfc750P_11-70002752/nwrfcsdk /usr/sap/nwrfcsdk
WORKDIR /usr/sap
RUN echo /usr/sap/nwrfcsdk/lib > /etc/ld.so.conf.d/nwrfcsdk.conf
RUN ldconfig

# Version of Gregor Kraliks SAP Netweaver RFC module source code for PHP 7
ARG SAPNWRFC_VERSION=1.4.0

# Download Gregor Kraliks SAP Netweaver RFC module source code for PHP 7
WORKDIR /usr/sap
RUN curl --insecure --fail --location --remote-name "https://github.com/gkralik/php7-sapnwrfc/archive/${SAPNWRFC_VERSION}.tar.gz"

# Extract the module sources.
RUN tar xzf ${SAPNWRFC_VERSION}.tar.gz
WORKDIR php7-sapnwrfc-${SAPNWRFC_VERSION}
RUN phpize
RUN ./configure --with-sapnwrfc=/usr/sap/nwrfcsdk
RUN make && make install

#RUN docker-php-ext-enable sapnwrfc

WORKDIR /var/www



