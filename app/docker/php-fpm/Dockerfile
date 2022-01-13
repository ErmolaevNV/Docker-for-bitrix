ARG BUILDKIT_INLINE_CACHE
ARG PHP_BASEIMAGE_VERSION

FROM mlocati/php-extension-installer:latest as local-php-extension-installer

FROM php:${PHP_BASEIMAGE_VERSION}-fpm-alpine as base
ARG PHP_COMPOSER_VERSION
ARG LOCAL_COMPOSER_REPOSITORY_URL
ARG TZ
ARG USER_ID
ARG GROUP_ID
ENV TZ=${TZ}
COPY --from=local-php-extension-installer /usr/bin/install-php-extensions /usr/bin/
RUN apk update && \
    apk add msmtp shadow tzdata && \
    install-php-extensions gd intl mysqli pdo_mysql pdo_sqlite soap xsl zip xdebug opcache memcached memcache redis gmp sockets pcntl @composer-${PHP_COMPOSER_VERSION}
RUN cp /usr/share/zoneinfo/${TZ} /etc/localtime && echo ${TZ} > /etc/timezone && \
    usermod -u ${USER_ID} www-data && groupmod -g ${GROUP_ID} www-data && \
    apk del shadow && \
    rm -rf /var/cache/apk/*
RUN touch $PHP_INI_DIR/conf.d/00-base.ini && \
    echo "date.timezone=${TZ}" >> $PHP_INI_DIR/conf.d/00-base.ini && \
    echo "expose_php=false" >> $PHP_INI_DIR/conf.d/00-base.ini

COPY ./docker/php-fpm/bitrix.ini $PHP_INI_DIR/conf.d/90-bitrix.ini
COPY ./docker/php-fpm/www.conf $PHP_INI_DIR/../php-fpm.d/www.conf
WORKDIR /app/
RUN chown www-data:www-data /app/

FROM base as develop
ARG XDEBUG_REMOTE_HOST
ARG MSMTPC_CONFIG
ADD ./docker/php-fpm/xdebug.ini $PHP_INI_DIR/conf.d/80-xdebug.ini
RUN echo "xdebug.client_host=${XDEBUG_REMOTE_HOST}" >> $PHP_INI_DIR/conf.d/80-xdebug.ini && \
    echo "${MSMTPC_CONFIG}" >> /etc/msmtprc && \
    cp "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

FROM base as production
ARG MSMTPC_CONFIG
COPY --chown=www-data:www-data ./local/composer.* ./local/
RUN composer --working-dir=local install --no-dev && \
    chown www-data:www-data -R /app/local
COPY --from=front --chown=www-data:www-data /app/ .
RUN rm -rf ./docker/ && \
    echo "${MSMTPC_CONFIG}" >> /etc/msmtprc && \
    rm $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini && \
    cp "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
