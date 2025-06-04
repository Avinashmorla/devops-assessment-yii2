# Use an official PHP image with Apache
FROM php:8.2-apache

# Install Composer and PHP extensions required by Yii2
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    libzip-dev \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql gd zip

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory in the container
WORKDIR /var/www/html

# Copy the application code
COPY . /var/www/html

# Install Yii2 application dependencies via Composer
# This assumes your composer.json is at the root of your app
RUN composer install --no-dev --prefer-dist --optimize-autoloader

# Configure Apache document root to web/
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf \
    && sed -i 's!/var/www/html!/var/www/html/web!g' /etc/apache2/sites-available/000-default.conf \
    && a2enmod rewrite

# Expose port 80
EXPOSE 80
