---
title: "Docker php"
type: "docs"
weight: 2
---

```shell
# docker build -t register.xxx.com/my/alpine:php7.2 -f ./Dockerfile .
# docker build -t register.xxx.com/my/alpine:php7.4 -f ./Dockerfile .
# docker buildx build --platform linux/amd64,linux/arm64 -t register.xxx.com/my/alpine:php7.4 .
# docker push register.xxx.com/cp/alpine:php7.4

# php:7.x-fpm-alpine安装扩展不方便不建议使用
# alpine:3.9 php7(7.2.33)
#FROM alpine:3.9
# alpine:3.15 php7(7.4.33)、php8(8.0.25)
FROM alpine:3.15

# 时区
ENV TZ=Asia/Shanghai

# 阿里云镜像（全国推荐）
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
# 阿里云内网
#RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.cloud.aliyuncs.com/g' /etc/apk/repositories
# 清华大学镜像（北方推荐）
#RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
# 中国科学技术大学镜像（南方推荐）
#RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

# timezone、crontab、supervisor、php
RUN apk add --no-cache \
    tzdata \
#    dcron \
#    supervisor \
    php7 \
    php7-fpm \
    php7-imap \
    php7-phar \
    php7-zip \
    php7-zlib \
    php7-sockets \
    php7-dom \
    php7-fileinfo \
    php7-simplexml \
    php7-xml \
    php7-xmlreader \
    php7-xmlwriter \
    php7-json \
    php7-tokenizer \
    php7-pcntl \
    php7-posix \
    php7-mbstring \
    php7-bcmath \
    php7-pdo_mysql \
    php7-mongodb \
    php7-pecl-redis \
    php7-pecl-rdkafka \
    php7-curl \
    php7-iconv \
    php7-gd

# composer
#RUN wget https://getcomposer.org/composer-stable.phar -O /usr/local/bin/composer && chmod +x /usr/local/bin/composer
RUN wget https://mirrors.aliyun.com/composer/composer.phar -O /usr/local/bin/composer && chmod +x /usr/local/bin/composer
RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

# 创建用户
#RUN addgroup nonroot && adduser -G nonroot -D nonroot
# 切换用户
#USER nobody
# 或者使用宿主机用户
#ARG uid=1000
#ARG gid=1000
#RUN addgroup -g ${gid} test && adduser -u ${uid} -G test -D test

# 工作目录
WORKDIR /code/

#composer install
#supervisord -c /etc/supervisord.conf -n

# PHP-FPM
EXPOSE 9000

ENTRYPOINT php-fpm7 --nodaemonize
```
