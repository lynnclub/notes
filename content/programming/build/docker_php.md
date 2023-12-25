---
title: "Docker php"
type: "docs"
weight: 3
---

```shell
FROM alpine:3.15

# 阿里云镜像（全国推荐）
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
# 阿里云内网
#RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.cloud.aliyuncs.com/g' /etc/apk/repositories
# 清华镜像（北方推荐，阿里云国内构建节点位于北京）
#RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
# 中科大镜像（南方推荐）
#RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

# 时区
RUN apk add --no-cache tzdata
ENV TZ=Asia/Shanghai

# 安装php-fpm
RUN apk add --no-cache php7 php7-fpm php7-imap php7-phar php7-zip php7-zlib php7-sockets php7-dom php7-fileinfo php7-simplexml php7-xml php7-xmlreader php7-xmlwriter php7-json php7-tokenizer php7-pcntl php7-posix php7-mbstring php7-bcmath php7-pdo_mysql php7-mongodb php7-pecl-redis php7-gd php7-curl

# 安装composer
RUN curl -L -O https://mirrors.aliyun.com/composer/composer.phar
RUN chmod +x composer.phar && mv composer.phar /usr/local/bin/composer
RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

# 安装supervisor
RUN apk add --no-cache supervisor

# 用户git
RUN addgroup git && adduser -G git -D git

# PHP-FPM
EXPOSE 9000

# 工作目录
WORKDIR /usr/src/

ENTRYPOINT php-fpm7 --nodaemonize
```
