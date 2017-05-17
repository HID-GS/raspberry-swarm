FROM armhf/php:7.1-fpm

# install the PHP extensions we need
RUN set -ex \
	&& buildDeps=' \
		libjpeg62-turbo-dev \
		libpng12-dev \
		libpq-dev \
	' \
	&& apt-get update && apt-get install -y --no-install-recommends $buildDeps && rm -rf /var/lib/apt/lists/* \
	&& docker-php-ext-configure gd \
		--with-jpeg-dir=/usr \
		--with-png-dir=/usr \
	&& docker-php-ext-install -j "$(nproc)" gd mbstring opcache pdo pdo_mysql pdo_pgsql zip \
# PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/local/lib/php/extensions/no-debug-non-zts-20151012/gd.so' - libjpeg.so.62: cannot open shared object file: No such file or directory in Unknown on line 0
# PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/local/lib/php/extensions/no-debug-non-zts-20151012/pdo_pgsql.so' - libpq.so.5: cannot open shared object file: No such file or directory in Unknown on line 0
	&& apt-mark manual \
		libjpeg62-turbo \
		libpq5 \
	&& apt-get purge -y --auto-remove $buildDeps

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
		echo 'memory_limit=512M'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# set develop PHP.ini settings
RUN { \
		echo 'error_log=/var/log/php_errors.log'; \
		echo 'log_errors=1'; \
		echo 'error_reporting=E_ALL'; \
		echo 'auto_detect_line_endings=1'; \
		echo 'max_execution_time = 300'; \
		echo 'upload_max_filesize = 50M'; \
		echo 'post_max_size = 50M'; \
	} > /usr/local/etc/php/conf.d/develop-settings.ini

# Composer
ARG INSTALL_COMPOSER=true
RUN if [ ${INSTALL_COMPOSER} = true ]; then \
		php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
		&& php -r "if (hash_file('SHA384', 'composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
		&& php composer-setup.php \
		&& php -r "unlink('composer-setup.php');" \
		&& mv composer.phar /usr/local/bin/composer \
;fi

# Drupal console
ARG INSTALL_DRUPAL_CLI=false
RUN if [ ${INSTALL_DRUPAL_CLI} = true ]; then \
		php -r "readfile('https://drupalconsole.com/installer');" > drupal.phar \
		&& mv drupal.phar /usr/local/bin/drupal \
		&& chmod +x /usr/local/bin/drupal \
;fi

RUN mkdir -p /var/www/drupal
WORKDIR /var/www/drupal
RUN usermod -u 82 www-data
EXPOSE 9000

CMD ["php-fpm"]
