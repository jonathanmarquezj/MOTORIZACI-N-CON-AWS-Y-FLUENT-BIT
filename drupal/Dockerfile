# IMAGEN BASE
FROM drupal:8.9.13-php7.4-fpm-buster as base


## PAQUETES NECESARIOS
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    unzip \
    git \
    vim \
    sudo \
    openssh-server \
    rsync \
    default-mysql-client \
	libfcgi-bin \
    openssh-client && \
    apt-get clean -y && \
    apt-get autoclean -y && \
    apt-get autoremove -y && \
    rm -rf /var/lib/{apt,dpkg,cache,log}/ && \
    rm -rf /var/lib/apt/lists/*

# DIRECTORIO EN EL CUAL NOS REDIGIREMOS AL CONECTARNOS EN EL CONTENEDOR
WORKDIR /var/www/html

# PUERTO DEL CONTENEDOR
EXPOSE 9000


CMD ["php-fpm","--fpm-config","/usr/local/etc/php-fpm.conf","-c","/usr/local/etc/php/php.ini"]