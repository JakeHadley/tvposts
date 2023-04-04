FROM php:8.1-rc-apache

RUN a2enmod rewrite
RUN a2enmod http2

#Uncoment for development enviroment
ADD certs/ssl.crt /etc/apache2/ssl/ssl.crt
ADD certs/ssl.key /etc/apache2/ssl/ssl.key
ADD ./configFiles/default /etc/apache2/sites-available/000-default.conf

RUN echo "Running development enviroment" \
    && a2enmod ssl \
    && mkdir -p /var/run/apache2/ \
    && mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

# install the PHP extensions we need
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    unzip \
    sudo \
    libgmp-dev \
    libxml2-dev \
    git \
    default-mysql-client \
    vim \
    libzip-dev \
    zip \
    net-tools \
    iputils-ping \
    tzdata \
    && rm -rf /var/lib/apt/lists/* \
	&& ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/include/gmp.h \
	#&& docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \

	&& docker-php-ext-install gd soap opcache mysqli pdo pdo_mysql gmp zip \
	&& curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN chown -R www-data:www-data /var/www/

# Configure
RUN pecl install xdebug && \
    docker-php-ext-enable xdebug

# We add the rigth xdebug config
ADD configFiles/xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
		} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# set timezone
RUN { \
		echo 'date.timezone=GMT+0'; \
	} > /usr/local/etc/php/conf.d/datetime.ini
# https://modx.com/download/direct/modx-3.0.1-pl.zip
# Download specified modx version ${MODX_VERSION}
USER www-data
RUN chown -R www-data:www-data /var/www && chmod -R 0777 /var/www

CMD ["apache2-foreground"]

