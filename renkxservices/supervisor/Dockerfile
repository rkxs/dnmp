ARG PHP_VERSION
FROM ${PHP_VERSION}

ARG TZ
ARG PHP_EXTENSIONS
ARG CONTAINER_PACKAGE_URL


RUN if [ $CONTAINER_PACKAGE_URL ] ; then sed -i "s/dl-cdn.alpinelinux.org/${CONTAINER_PACKAGE_URL}/g" /etc/apk/repositories ; fi


COPY ./services/php/extensions /tmp/extensions
WORKDIR /tmp/extensions
RUN chmod +x install.sh \
    && sh install.sh \
    && rm -rf /tmp/extensions

ADD ./services/php/extensions/install-php-extensions  /usr/local/bin/

RUN chmod uga+x /usr/local/bin/install-php-extensions

RUN apk --no-cache add tzdata \
    && cp "/usr/share/zoneinfo/$TZ" /etc/localtime \
    && echo "$TZ" > /etc/timezone


# Fix: https://github.com/docker-library/php/issues/240
RUN apk add gnu-libiconv libstdc++ --no-cache --repository http://${CONTAINER_PACKAGE_URL}/alpine/edge/community/ --allow-untrusted
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php


# Install composer and change it's cache home todo supervisor服务不需要composer
#RUN curl -o /usr/bin/composer https://mirrors.aliyun.com/composer/composer.phar \
#    && chmod +x /usr/bin/composer
#ENV COMPOSER_HOME=/tmp/composer

# php image's www-data user uid & gid are 82, change them to 1000 (primary user)
RUN apk --no-cache add shadow && usermod -u 1000 www-data && groupmod -g 1000 www-data

# todo 安装supervisor
RUN apk update \
	&& apk upgrade \
	&& apk --no-cache add supervisor openssh \
	&& rm -rf /var/cache/apk/* \
    # todo 更改用户shell www-data需要执行定时脚本
    && usermod -s /bin/ash www-data

# todo 设置挂载
VOLUME ["/etc/supervisor/conf.d", "/var/log/supervisor/"]

# todo 工作目录
WORKDIR /etc/supervisor/conf.d

CMD ["supervisord", "--nodaemon", "--configuration", "/etc/supervisor/supervisord.conf"]