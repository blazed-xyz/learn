FROM php:8.0-apache
#WORKDIR /var/www/html

# Setup Composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

COPY index.php /var/www/html/index.php
#COPY src/ src
EXPOSE 80