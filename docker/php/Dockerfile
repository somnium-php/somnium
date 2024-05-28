ARG UBUNTU_VERSION

FROM ubuntu:$UBUNTU_VERSION AS release

ARG PHP_VERSION
ARG NON_ROOT_USER=somnium
ARG DEBIAN_FRONTEND=noninteractive

ENV PHP_VERSION=$PHP_VERSION
ENV PHP_DIR=/etc/php/$PHP_VERSION
ENV NON_ROOT_USER=$NON_ROOT_USER

RUN \
    #---------------------------------------------------------------------------
    # Update System Packages
    #---------------------------------------------------------------------------
        apt update \
        && apt upgrade -y \
    #---------------------------------------------------------------------------
    # Add Non-Root User/Group
    #---------------------------------------------------------------------------
        && id -un 1000 | xargs -r userdel -frRZ \
        && groupadd -g 1000 $NON_ROOT_USER \
        && useradd -u 1000 -g 1000 -m $NON_ROOT_USER -s /bin/bash \
        && usermod -L $NON_ROOT_USER \
    #---------------------------------------------------------------------------
    # Install PHP
    #---------------------------------------------------------------------------
        && apt install -y software-properties-common \
        && add-apt-repository -y ppa:ondrej/php \
        && apt update \
        && apt install -y php$PHP_VERSION \
    #---------------------------------------------------------------------------
    # Install and Configure PHP-FPM
    #---------------------------------------------------------------------------
        && apt install -y php$PHP_VERSION-fpm \
        && sed -i "s/^\(\s*\);*\s*\(error_log\)\s*=\s*.*/\1\2 = \/dev\/stderr/" $PHP_DIR/fpm/php-fpm.conf \
        && sed -i "s/^\(\s*\);*\s*\(log_limit\)\s*=\s*.*/\1\2 = 16384/" $PHP_DIR/fpm/php-fpm.conf \
        && mkdir -p /run/php \
        && touch /run/php/php$PHP_VERSION-fpm.sock \
    #---------------------------------------------------------------------------
    # Install PHP Extensions
    #---------------------------------------------------------------------------
        && apt install -y \
            php$PHP_VERSION-bcmath \
            php$PHP_VERSION-cli \
            php$PHP_VERSION-curl \
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
        && echo '* * * * * cd /var/www && php aspire schedule:run >> /dev/null 2>&1' | crontab -u $NON_ROOT_USER - \
    #---------------------------------------------------------------------------
    # Post-Installation Cleanup
    #---------------------------------------------------------------------------
        && apt autoremove -y --purge \
        && apt clean \
        && rm -rf /var/lib/apt/lists/* /var/tmp/*

COPY php-custom.ini $PHP_DIR/cli/conf.d/
COPY php-custom.ini $PHP_DIR/fpm/conf.d/

COPY pool.d/ $PHP_DIR/fpm/pool.d/

WORKDIR /var/www

CMD /usr/sbin/php-fpm$PHP_VERSION --nodaemonize





FROM release AS develop

ARG DEBIAN_FRONTEND=noninteractive

RUN \
    #---------------------------------------------------------------------------
    # Update System Packages
    #---------------------------------------------------------------------------
        apt update \
    #---------------------------------------------------------------------------
    # Install Development Packages
    #---------------------------------------------------------------------------
        && apt install -y \
            curl \
            dnsutils \
            iputils-ping \
            nano \
            php$PHP_VERSION-xdebug \
            vim \
    #---------------------------------------------------------------------------
    # Post-Installation Cleanup
    #---------------------------------------------------------------------------
        && apt autoremove -y --purge \
        && apt clean \
        && rm -rf /var/lib/apt/lists/* /var/tmp/*

COPY xdebug.ini $PHP_DIR/mods-available/
