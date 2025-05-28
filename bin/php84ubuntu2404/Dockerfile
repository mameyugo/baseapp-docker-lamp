FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

# Requisitos básicos
RUN apt update && apt install -y \
    software-properties-common gnupg2 curl unzip git g++ make nano openssh-server lsb-release ca-certificates \
    apache2 mariadb-client libapache2-mod-php \
    && add-apt-repository ppa:ondrej/php \
    && apt update && apt install -y \
    php8.4 php8.4-bcmath php8.4-calendar php8.4-ctype php8.4-curl php8.4-date php8.4-exif \
    php8.4-ffi php8.4-fileinfo php8.4-filter php8.4-ftp php8.4-gd php8.4-gettext php8.4-iconv \
    php8.4-json php8.4-mbstring php8.4-mysqli php8.4-mysql php8.4-openssl php8.4-pdo \
    php8.4-pdo-mysql php8.4-pdo-sqlite php8.4-phar php8.4-readline php8.4-reflection php8.4-session \
    php8.4-simplexml php8.4-soap php8.4-sqlite3 php8.4-tokenizer php8.4-xml php8.4-xmlreader \
    php8.4-xmlwriter php8.4-xsl php8.4-zip php8.4-zlib php8.4-dom php8.4-bz2 php8.4-ldap \
    php8.4-pspell php8.4-intl php8.4-oci8 php8.4-common \
    php8.4-dev php8.4-cli php8.4-opcache \
    && apt install -y libzip-dev libpng-dev libonig-dev libxml2-dev libldap2-dev libpq-dev libicu-dev \
    && apt clean

# install xdebug
RUN pecl install xdebug && docker-php-ext-enable xdebug
RUN echo "xdebug.mode=debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Habilitar mod_rewrite
RUN a2enmod rewrite

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

# Psalm y PHPUnit
RUN composer global require vimeo/psalm phpunit/phpunit --no-progress --no-suggest

# Agregar Composer global al PATH
ENV PATH="/root/.composer/vendor/bin:${PATH}"

# Copiar configuración php.ini
COPY php.ini /etc/php/8.4/apache2/php.ini

# Apache web root
WORKDIR /var/www/html

# Fix Permissions
RUN usermod -u 1000 www-data

EXPOSE 80 22

CMD service ssh start && apache2ctl -D FOREGROUND
# Volumen para el código fuente
VOLUME ["/var/www/html"]