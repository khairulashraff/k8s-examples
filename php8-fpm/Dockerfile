# # 
# Separate stage for building process as we don't need the tools when running the app.
# We actually don't need FPM here but we do need it later, so to not waste downloading
# so many images, we'll just use the FPM image anyway.
# #
FROM php:8-fpm-alpine as build

WORKDIR /app

# ! IMPORTANT !
# This setup process was done with Laravel 9 in mind. You may need to change it
# according to your usage.

# Get composer and node/npm
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
  php composer-setup.php --install-dir /usr/local/bin --filename=composer
RUN apk add --update npm

# Don't change this though. It's sandwiched here for better caching.
COPY ./app /app

# Run setup/build process for your app
RUN composer install
RUN npm i && npm run production

# #
# Running image with FPM.
# For the most part, nothing beyond here needs changing except if you want to add more PHP extensions.
# #
FROM php:8-fpm-alpine as final

# Setup document root
WORKDIR /var/www/html

# Install docker-php-extension-installer to get more extensions
# Check the repo for a list of supported extensions.
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod +x /usr/local/bin/install-php-extensions && \
    install-php-extensions pdo_mysql

# Install packages and remove default server definition
RUN apk add --no-cache \
  curl \
  nginx \
  supervisor

# Make sure files/folders needed by the processes are accessable when they run under the www-data user
RUN chown -R www-data.www-data /var/www/html /run /var/lib/nginx /var/log/nginx

# Configure nginx / supervisord
COPY ./configs/nginx.conf /etc/nginx/nginx.conf
COPY ./configs/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./configs/fpm-pool.conf /usr/local/etc/php-fpm.d/www.conf

# We're done installing, go back down to www-data
USER www-data

# Copy built app from build stage
COPY --chown=www-data --from=build /app /var/www/html/

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping
