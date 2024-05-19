ARG UBUNTU_VERSION

FROM ubuntu:$UBUNTU_VERSION AS release

ARG PHP_VERSION
ARG NON_ROOT_USER=somnium
ARG DEBIAN_FRONTEND=noninteractive

ENV PHP_VERSION=$PHP_VERSION
ENV PHP_DIR=/etc/php/$PHP_VERSION

RUN \
    #---------------------------------------------------------------------------
    # Update System Packages
    #---------------------------------------------------------------------------
        apt update \
        && apt upgrade -y \
    #---------------------------------------------------------------------------
    # Add Default User/Group
    #---------------------------------------------------------------------------
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
        && sed -i "s/^\s*;*\s*\(error_log\)\s*=\s*.*/\1 = \/proc\/self\/fd\/2/" $PHP_DIR/fpm/php-fpm.conf \
        && sed -i "s/^\s*;*\s*\(log_limit\)\s*=\s*.*/\1 = 8192/" $PHP_DIR/fpm/php-fpm.conf \
        && sed -i "s/^\s*;*\s*\(user\)\s*=\s*.*/\1 = $NON_ROOT_USER/" $PHP_DIR/fpm/pool.d/www.conf \
        && sed -i "s/^\s*;*\s*\(group\)\s*=\s*.*/\1 = $NON_ROOT_USER/" $PHP_DIR/fpm/pool.d/www.conf \
        && sed -i "s/^\s*;*\s*\(listen\)\s*=\s*.*/\1 = 0.0.0.0:9000/" $PHP_DIR/fpm/pool.d/www.conf \
        && sed -i "s/^\s*;*\s*\(listen\.owner\)\s*=\s*.*/\1 = $NON_ROOT_USER/" $PHP_DIR/fpm/pool.d/www.conf \
        && sed -i "s/^\s*;*\s*\(listen\.group\)\s*=\s*.*/\1 = $NON_ROOT_USER/" $PHP_DIR/fpm/pool.d/www.conf \
        && sed -i "s/^\s*;*\s*\(access\.log\)\s*=\s*.*/\1 = \/proc\/self\/fd\/2/" $PHP_DIR/fpm/pool.d/www.conf \
        && sed -i "s/^\s*;*\s*\(catch_workers_output\)\s*=\s*.*/\1 = yes/" $PHP_DIR/fpm/pool.d/www.conf \
        && sed -i "s/^\s*;*\s*\(clear_env\)\s*=\s*.*/\1 = no/" $PHP_DIR/fpm/pool.d/www.conf \
        && sed -i "s/^\s*;*\s*\(decorate_workers_output\)\s*=\s*.*/\1 = no/" $PHP_DIR/fpm/pool.d/www.conf \
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
