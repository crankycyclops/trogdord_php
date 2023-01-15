# This docker file is exactly the same as the official php:8.1-fpm container,
# but adds to it the trogdord extension.

# To build for 8.0: docker build --build-arg PHP_VER=8.0 --build-arg MODULE_VER=<version> -t crankycyclops/php8.0-fpm-trogdord-ext:<version> .
# To build for 8.1: docker build --build-arg PHP_VER=8.1 --build-arg MODULE_VER=<version> -t crankycyclops/php8.1-fpm-trogdord-ext:<version> .
# To build for 8.2: docker build --build-arg PHP_VER=8.2 --build-arg MODULE_VER=<version> -t crankycyclops/php8.2-fpm-trogdord-ext:<version> .

# To build a version that includes npm (required to bootstrap newly installed Laraval applications):

# docker build --build-arg PHP_VER=8.0 --build-arg INCLUDE_NPM=true --build-arg MODULE_VER=<version> -t crankycyclops/php8.0-fpm-trogdord-ext-npm:<version> .
# docker build --build-arg PHP_VER=8.1 --build-arg INCLUDE_NPM=true --build-arg MODULE_VER=<version> -t crankycyclops/php8.1-fpm-trogdord-ext-npm:<version> .
# docker build --build-arg PHP_VER=8.2 --build-arg INCLUDE_NPM=true --build-arg MODULE_VER=<version> -t crankycyclops/php8.2-fpm-trogdord-ext-npm:<version> .

# If you connect to the container and can't run npm, execute the following command first:
# source /etc/profile
# I've noticed that this gets borked when I set DOCKER_UID. Oh well.

# Determines which version of PHP we're building the container for
ARG PHP_VER=8.2

# Determines whether or not we add support for npm (disabled by default)
ARG INCLUDE_NPM=false

FROM php:${PHP_VER}-fpm

# See: https://stackoverflow.com/a/56748289
ARG INCLUDE_NPM

# Required: tells us which version of the repository to build with
ARG MODULE_VER
RUN test -n "$MODULE_VER"
ENV MODULE_VER=${MODULE_VER}

RUN apt-get update
RUN apt-get -y install g++ wget rapidjson-dev libasio-dev libzip4 libzip-dev

WORKDIR /usr/local/src
RUN wget https://github.com/crankycyclops/trogdord_php/archive/refs/tags/$MODULE_VER.tar.gz
RUN tar -zvxpf $MODULE_VER.tar.gz
WORKDIR /usr/local/src/trogdord_php-$MODULE_VER

RUN phpize && ./configure && make -j$(nproc) && make install
RUN docker-php-ext-enable trogdord
RUN docker-php-ext-install bcmath zip pdo pdo_mysql

# Install composer
WORKDIR /usr/bin
RUN wget -c https://getcomposer.org/download/latest-stable/composer.phar
RUN mv composer.phar composer
RUN chmod 755 composer

# Optionally install npm
ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION 16
RUN if [ "true" = "$INCLUDE_NPM" ]; then \
    mkdir /usr/local/nvm; \
    wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash; \
    . $NVM_DIR/nvm.sh; \
    nvm install $NODE_VERSION; \
    nvm alias default $NODE_VERSION; \
    nvm use default; \
    echo "source /usr/local/nvm/nvm.sh" >> /etc/profile; \
fi

# Cleanup after building
RUN apt-get remove -y g++ wget rapidjson-dev libasio-dev
RUN apt-get autoremove -y
RUN rm -rf /usr/local/src/trogdord_php-$MODULE_VER

# Without this, we won't be able to find the npm command when connecting to a running container via bash
ENV BASH_ENV="/etc/profile"

WORKDIR /
ENTRYPOINT ["php-fpm"]
