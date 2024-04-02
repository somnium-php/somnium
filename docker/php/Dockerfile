ARG UBUNTU_VERSION

FROM ubuntu:${UBUNTU_VERSION} AS release

ARG PHP_VERSION
ARG DEBIAN_FRONTEND=noninteractive

ENV PHP_VERSION=${PHP_VERSION}
ENV PHP_DIR=/etc/php/${PHP_VERSION}

RUN \
    #---------------------------------------------------------------------------
    # Update System Packages
    #---------------------------------------------------------------------------
        apt update \
        && apt upgrade -y \
    #---------------------------------------------------------------------------
    # Add Default User/Group
    #---------------------------------------------------------------------------
        && groupadd -g 1000 somnium \
        && useradd -u 1000 -g 1000 -m somnium -s /bin/bash \
        && usermod -L somnium \
    #---------------------------------------------------------------------------
    # Install PHP
    #---------------------------------------------------------------------------
        && apt install -y software-properties-common \
        && add-apt-repository -y ppa:ondrej/php \
        && apt update \
        && apt install -y php${PHP_VERSION} \
    #---------------------------------------------------------------------------
    # Install and Configure PHP-FPM
    #---------------------------------------------------------------------------
        && apt install -y php${PHP_VERSION}-fpm \
        && sed -i 's/^\(\s*user\s*=\s*\).*/\1somnium/' ${PHP_DIR}/fpm/pool.d/www.conf \
        && sed -i 's/^\(\s*group\s*=\s*\).*/\1somnium/' ${PHP_DIR}/fpm/pool.d/www.conf \
        && sed -i 's/^\(\s*listen\s*=\s*\).*/\10.0.0.0:9000/' ${PHP_DIR}/fpm/pool.d/www.conf \
        && sed -i 's/^\(\s*listen\.owner\s*=\s*\).*/\1somnium/' ${PHP_DIR}/fpm/pool.d/www.conf \
        && sed -i 's/^\(\s*listen\.group\s*=\s*\).*/\1somnium/' ${PHP_DIR}/fpm/pool.d/www.conf \
        && mkdir -p /run/php \
        && touch /run/php/php${PHP_VERSION}-fpm.sock \
    #---------------------------------------------------------------------------
    # Install PHP Extensions
    #---------------------------------------------------------------------------
        && apt install -y \
            php${PHP_VERSION}-bcmath \
            php${PHP_VERSION}-cli \
            php${PHP_VERSION}-curl \
    #---------------------------------------------------------------------------
    # Install Composer
    #---------------------------------------------------------------------------
        && apt install -y curl unzip \
        && curl -sS https://getcomposer.org/installer -o composer-setup.php \
        && apt remove -y curl \
        && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
        && rm composer-setup.php \
    #---------------------------------------------------------------------------
    # Install and Configure Task Scheduler
    #---------------------------------------------------------------------------
        && apt install -y cron \
        && echo '* * * * * cd /var/www && php console schedule:run >> /dev/null 2>&1' | crontab -u somnium - \
    #---------------------------------------------------------------------------
    # Post-Installation Cleanup
    #---------------------------------------------------------------------------
        && apt autoremove -y --purge \
        && apt clean \
        && rm -rf /var/lib/apt/lists/* /var/tmp/*

COPY php-custom.ini ${PHP_DIR}/cli/conf.d/
COPY php-custom.ini ${PHP_DIR}/fpm/conf.d/

WORKDIR /var/www

CMD /usr/sbin/php-fpm${PHP_VERSION} --nodaemonize





FROM release AS develop

ARG DEBIAN_FRONTEND=noninteractive

RUN \
    #---------------------------------------------------------------------------
    # Update System Packages
    #---------------------------------------------------------------------------
        apt update \
    #---------------------------------------------------------------------------
    # Install PHP Extensions
    #---------------------------------------------------------------------------
        && apt install -y php${PHP_VERSION}-xdebug \
    #---------------------------------------------------------------------------
    # Post-Installation Cleanup
    #---------------------------------------------------------------------------
        && apt autoremove -y --purge \
        && apt clean \
        && rm -rf /var/lib/apt/lists/* /var/tmp/*

COPY xdebug.ini ${PHP_DIR}/mods-available/
