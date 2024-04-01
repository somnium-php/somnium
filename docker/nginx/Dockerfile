ARG UBUNTU_VERSION

FROM ubuntu:${UBUNTU_VERSION} AS release

ARG DEBIAN_FRONTEND=noninteractive

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
    # Install and Configure Nginx
    #---------------------------------------------------------------------------
        && apt install -y nginx \
        && sed -i 's/^\(\s*user\s\).*;/\1somnium;/' /etc/nginx/nginx.conf \
        && sed -i 's/^\(\s*\)#\s*gzip\(\s\|_\)/\1gzip\2/' /etc/nginx/nginx.conf \
        && ln -sf /dev/stdout /var/log/nginx/access.log \
        && ln -sf /dev/stderr /var/log/nginx/error.log \
    #---------------------------------------------------------------------------
    # Post-Installation Cleanup
    #---------------------------------------------------------------------------
        && apt autoremove -y --purge \
        && apt clean \
        && rm -rf /var/lib/apt/lists/* /var/tmp/*

COPY default.conf /etc/nginx/sites-available/default

WORKDIR /var/www

EXPOSE 80/tcp

CMD ["nginx", "-g", "daemon off;"]
