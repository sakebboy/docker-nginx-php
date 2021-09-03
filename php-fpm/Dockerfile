FROM php:7.2-fpm
RUN apt-get update && apt-get install iputils-ping -y
RUN pecl install xdebug-2.6.0 && docker-php-ext-enable xdebug

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
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* \
	&& rm /var/log/lastlog /var/log/faillog

# Add Microsoft repo for Microsoft ODBC Driver 13 for Linux
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/8/prod.list > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update

# Install Dependencies
RUN ACCEPT_EULA=Y apt-get install -y unixodbc
RUN ACCEPT_EULA=Y apt-get install -y unixodbc-dev
RUN ACCEPT_EULA=Y apt-get install -y libgss3
RUN ACCEPT_EULA=Y apt-get install -y odbcinst
RUN ACCEPT_EULA=Y apt-get install -y locales
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen
		
#composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
	
# Install required extensions
RUN pecl install apcu-5.1.5 pdo_sqlsrv-5.8.1 sqlsrv-5.8.1 
RUN docker-php-ext-enable apcu
RUN docker-php-ext-enable sqlsrv
RUN docker-php-ext-enable pdo_sqlsrv
RUN docker-php-ext-install intl
RUN docker-php-ext-install mysqli
RUN docker-php-ext-install pdo
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-install mbstring
RUN docker-php-ext-install bcmath
RUN docker-php-ext-install soap
RUN docker-php-ext-install zip
RUN docker-php-ext-install opcache
RUN docker-php-ext-install gd

# 逐步安裝避免出現Cannot find ldap.h
RUN apt-get install libldap2-dev -y \ 
	&& rm -rf /var/lib/apt/lists/* \
	&& docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
	&& docker-php-ext-install ldap


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

#RUN docker-php-ext-configure mysqli \
#	&& docker-php-ext-install -j$(nproc) mysqli \
#	&& docker-php-ext-configure gd \
#	&& docker-php-ext-install -j$(nproc) gd \
#	&& docker-php-ext-configure ldap \
#	&& docker-php-ext-configure sockets \
#	&& docker-php-ext-install -j$(nproc) sockets \
#	&& docker-php-ext-configure pdo_mysql \
#	&& docker-php-ext-install -j$(nproc) pdo_mysql \
#	&& docker-php-ext-configure zip \
#	&& docker-php-ext-install -j$(nproc) zip \


