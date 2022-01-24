##
# dev stage
##
FROM php:8.0.3-fpm as dev

RUN apt-get update && \
    apt-get install -y zlib1g-dev \
                       g++ \
                       curl \
                       git \
                       libicu-dev \
                       libzip-dev \
                       unzip && \
    docker-php-ext-install intl \
                           opcache \
                           pdo \
                           pdo_mysql && \
    pecl install xdebug-3.0.3 apcu && \
    docker-php-ext-enable xdebug apcu && \
    docker-php-ext-configure zip && \
    docker-php-ext-install zip

# Set application directory
RUN rm -r /var/www/html && \
    mkdir /var/www/app

WORKDIR /var/www/app

# Add some convenience
RUN echo "alias ll='ls -al --color'" > /etc/profile.d/app.sh
RUN echo "alias bc='php bin/console'" | tee -a /etc/profile.d/app.sh

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Use php.ini for development
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

# Set timezone
RUN rm /etc/localtime
RUN ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime
RUN "date"

# Install symfony installer
RUN curl -sS https://get.symfony.com/cli/installer | bash
RUN mv /root/.symfony/bin/symfony /usr/local/bin/symfony
RUN git config --global user.email "olaf.helms@adesso.de" && \
    git config --global user.name "Olaf Helms"

# Install Node.js and Yarn
RUN apt-get --quiet update  && \
    apt-get --quiet install --yes gnupg \
                                  gnupg2
ENV NODE_VERSION=12.x
RUN set -xe && \
    # Node.js repo
    curl -sSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - && \
    echo "deb https://deb.nodesource.com/node_${NODE_VERSION} jessie main" | tee /etc/apt/sources.list.d/nodesource.list && \
    echo "deb-src https://deb.nodesource.com/node_${NODE_VERSION} jessie main" | tee -a /etc/apt/sources.list.d/nodesource.list && \
    # Yarn repo
    curl -fsSL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
    apt-get update && \
    apt-get install --quiet --yes nodejs yarn --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*

##
# prod stage
##
FROM dev as prod

## Use php.ini for production
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

# Disable xdebug in production
RUN rm /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Install dependencies
ADD composer.json composer.lock ./
RUN composer install --no-interaction --no-dev --no-scripts --no-plugins

# Add app code
ADD . ./

CMD ["sh", "/var/www/app/docker-entrypoint.sh"]
