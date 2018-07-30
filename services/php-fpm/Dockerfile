FROM ubuntu:bionic

ARG UNAME=appuser
ARG UID=1000
ARG GID=1000

ARG PHP_VERSION="7.1"
ARG BLACKFIRE_VERSION="1.10.0"

RUN groupadd -g $GID $UNAME
RUN useradd -m -u $UID -g $GID -s /bin/bash $UNAME

RUN export DEBIAN_FRONTEND=noninteractive \
    && export LC_ALL='en_US.UTF-8' \
    && export LANG='en_US.UTF-8' \
    && export LANGUAGE='en_US.UTF-8' \
    && apt-get -qq update \
    && apt-get -qqy install --no-install-recommends \
      software-properties-common \
      curl \
      wget \
      gettext \
      git \
      nodejs \
      npm \
      bzip2 \
      locales \
      supervisor \
      nginx \
      gpg-agent \
    && localedef -c -f UTF-8 -i en_US en_US.UTF-8 \
    && locale-gen en en_US en_US.UTF-8 && dpkg-reconfigure locales \
    && wget -O - https://packagecloud.io/gpg.key | apt-key add - \
    && echo "deb http://packages.blackfire.io/debian any main" > /etc/apt/sources.list.d/blackfire.list \
    && add-apt-repository -y ppa:ondrej/php && apt-get -qq update \
    && apt-get -qqy install --no-install-recommends --fix-missing \
      php${PHP_VERSION}-fpm \
      php${PHP_VERSION}-cli \
      php${PHP_VERSION}-common \
      php${PHP_VERSION}-mysql \
      php${PHP_VERSION}-pgsql \
      php${PHP_VERSION}-curl \
      php${PHP_VERSION}-gd \
      php${PHP_VERSION}-mcrypt \
      php${PHP_VERSION}-xmlrpc \
      php${PHP_VERSION}-ldap \
      php${PHP_VERSION}-xsl \
      php${PHP_VERSION}-intl \
      php${PHP_VERSION}-soap \
      php${PHP_VERSION}-mbstring \
      php${PHP_VERSION}-zip \
      php${PHP_VERSION}-bz2 \
      php${PHP_VERSION}-tidy \
      php${PHP_VERSION}-bcmath \
      php${PHP_VERSION}-xdebug \
      blackfire-php \
    && wget -O /usr/local/bin/blackfire https://packages.blackfire.io/binaries/blackfire-agent/${BLACKFIRE_VERSION}/blackfire-cli-linux_static_amd64 \
    && chmod +x /usr/local/bin/blackfire \
    && apt-get -qy autoremove --purge software-properties-common \
    && apt-get autoclean

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN ln -sf /usr/sbin/php-fpm${PHP_VERSION} /usr/local/bin/php-fpm
RUN ln -sf /etc/php/${PHP_VERSION} /etc/php/current

ENV SYMFONY_ENV="prod"
ENV SYMFONY_DEBUG=0
ENV OPCACHE_ENABLED="1"
ENV XDEBUG_ENABLED="0"
ENV NPM_CONFIG_PREFIX=/usr/local/etc/npm

ENV NGINX_USER={UNAME}
ENV PHP_FPM_USER={UNAME}
ENV PHP_FPM_GROUP={UNAME}

COPY php/xdebug.ini    /etc/php/${PHP_VERSION}/mods-available/xdebug.ini
COPY php/blackfire.ini /etc/php/${PHP_VERSION}/mods-available/blackfire.ini
COPY php/php.fpm.ini   /etc/php/${PHP_VERSION}/fpm/php.ini
COPY php/php.cli.ini   /etc/php/${PHP_VERSION}/cli/php.ini
COPY php/fpm.conf      /etc/php/${PHP_VERSION}/fpm/php-fpm.conf
COPY php/fpm.www.conf  /etc/php/${PHP_VERSION}/fpm/pool.d/www.conf

COPY supervisor/supervisord.conf /etc/supervisord.conf

COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY nginx/symfony.conf /etc/nginx/conf.d/default.conf
#COPY nginx/websocket.conf /etc/nginx/conf.d/websocket.conf

VOLUME ["/var/www/html/application"]
WORKDIR "/var/www/html/application"

COPY nginx/index.html /var/www/html/application/public/index.html

# Expose Ports
EXPOSE 80

CMD ["supervisord", "-n", "-c", "/etc/supervisord.conf"]