FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

# Requisitos básicos
RUN apt update && apt install -y \
    software-properties-common gnupg2 curl unzip git g++ make nano openssh-server lsb-release ca-certificates \
    apache2 mariadb-client libapache2-mod-php \
    && add-apt-repository ppa:ondrej/php \
    && apt update && apt install -y \
    php8.3 php8.3-bcmath php8.3-common php8.3-curl \
    php8.3-gd \
    php8.3-mbstring php8.3-mysql \
    php8.3-readline \
    php8.3-soap php8.3-sqlite3 php8.3-xml \
    php8.3-xsl php8.3-zip php8.3-bz2 php8.3-ldap \
    php8.3-pspell php8.3-intl \
    php8.3-dev php8.3-cli php8.3-opcache \
    && apt install -y libzip-dev libpng-dev libonig-dev libxml2-dev libldap2-dev libpq-dev libicu-dev \
    && apt clean

# install xdebug
#RUN pecl install xdebug && docker-php-ext-enable xdebug
RUN pecl install xdebug \
 && echo "zend_extension=$(find /usr/lib/php/ -name xdebug.so)" >> /etc/php/8.3/apache2/php.ini
#RUN echo "xdebug.mode=debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
#RUN echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Habilitar mod_rewrite
RUN a2enmod rewrite
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Configuración de Apache
RUN sed -i 's/AllowOverride None/AllowOverride All/g' /etc/apache2/apache2.conf

# Enable apache ssl
RUN mkdir -p /etc/apache2/ssl/
RUN openssl req -new -newkey rsa:4096 -days 3650 -nodes -x509 -subj \
    "/C=../ST=...../L=..../O=..../CN=..." \
    -keyout /etc/apache2/ssl/ssl.key -out /etc/apache2/ssl/ssl.crt
RUN mkdir -p /var/run/apache2/
RUN a2enmod ssl \
    && a2ensite default-ssl \
    && c_rehash /etc/apache2/ssl/

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Node.js + npm
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - && apt install -y nodejs

# SSH
RUN mkdir /var/run/sshd
RUN echo 'root:root' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# Usa un DNS confiable para evitar errores de resolución
#RUN echo "nameserver 1.1.1.1" > /etc/resolv.conf

# Exporta el timeout como variable de entorno (300 segundos por defecto)
ENV COMPOSER_PROCESS_TIMEOUT=60

# Ejecuta el require sin flags obsoletos
RUN composer global require vimeo/psalm phpunit/phpunit --no-progress

# Agregar Composer global al PATH
ENV PATH="/root/.composer/vendor/bin:${PATH}"

# Copiar configuración php.ini
# COPY ./config/php/php.ini /etc/php/8.3/apache2/php_2.ini

# Apache web root
WORKDIR /var/www/html

# Fix Permissions
RUN usermod -u 1010 www-data

EXPOSE 80 22

CMD service ssh start && apache2ctl -D FOREGROUND
# Volumen para el código fuente
VOLUME ["/var/www/html"]