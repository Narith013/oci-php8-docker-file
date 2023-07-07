# oci-php8-docker-file

# Use the official PHP 8 base image with Apache
FROM php:8-apache

# Install necessary packages and libraries
RUN apt-get update && apt-get install -y \
    libaio1 \
    unzip \
    git \
    zlib1g-dev \
    libzip-dev \
    libpng-dev \
    libxml2-dev

# Install PHP extensions
RUN docker-php-ext-install \
    zip \
    gd \
    pdo \
    pdo_mysql \
    mysqli \
    soap

# Install Oracle Instant Client and OCI8 extension
ARG ORACLE_INSTANT_CLIENT_VERSION=21_4
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/bin/
RUN install-php-extensions oci8

# Enable Apache mod_rewrite
RUN a2enmod rewrite

# Set the document root to the public folder
ENV APACHE_DOCUMENT_ROOT /var/www/html/public
# RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
# RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

RUN sed -i "s#DocumentRoot /var/www/html#DocumentRoot $APACHE_DOCUMENT_ROOT#g" /etc/apache2/sites-available/000-default.conf && \
    sed -i "s#/var/www/html#$APACHE_DOCUMENT_ROOT#g" /etc/apache2/apache2.conf && \
    a2enmod rewrite

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set up Laravel
WORKDIR /var/www/html
COPY . /var/www/html/
RUN composer install --optimize-autoloader --no-dev
RUN chown -R www-data:www-data /var/www/html

CMD ["apache2-foreground"]
