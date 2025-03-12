---
title: 使用docker compose搭建wordpress博客
categories: 瞎折腾记录
tags:
  - wordpress
  - docker
  - 博客
abbrlink: 880ca5b7
date: 2024-02-27 16:41:04
---

## 引言

前几天嫖了华为云的HECS新用户优惠，69块钱就能上1c2G1M的服务器一年。手里刚好有ICP备案好的域名，就打算把原来部署在Netlify上的静态网站撤掉，把博客慢慢迁移到华为云上。

### 架构设计

考虑到这台HECS使用了新用户优惠，续费的时候不一定有这种价格，可以考虑使用docker compose构建架构以实现方便迁移。整个架构有三个容器，wordpress，mysql和nginx，其中wordpress使用php-fpm的版本。

<!-- more -->

## 前期准备

### 安装Docker

HECS服务器系统使用CentOS Stream 9。

#### 设置仓库

```bash
$ sudo dnf install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

选用阿里云的镜像源：

```bash
sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

如果你的服务器位于境外，使用官方镜像源也许更快：

```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

#### 安装 Docker Engine-Community

```bash
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

#### 启用docker服务

```bash
sudo systemctl enable docker --now
```

#### 测试

```bash

[root@hecs-166145 wordpress]# docker -v
Docker version 25.0.3, build 4debf41
[root@hecs-166145 wordpress]#
```

## 构建项目

项目的目录结构如下：

```text
.
├── compose.yml
├── data
│   ├── mysql
│   └── wordpress
├── nginx
│   └── wordpress.conf
└── tls_certs
    ├── ffdhe2048.txt
    ├── fullchain.pem
    └── privkey.pem
```

compose.yml是docker compose配置文件；data/目录里存放wordpress和mysql的数据文件；nginx/是nginx的配置目录；tls_certs/里存放了tls证书。

### 配置docker compose

`compose.yml`有如下内容：

```yaml
version: '3'
services:
  wordpress:
    image: wordpress:6.4.3-php8.3-fpm-alpine
    container_name: wordpress
    restart: always
    environment:
      WORDPRESS_DB_HOST: 'mysql'
      WORDPRESS_DB_USER: 'wdpress'
      WORDPRESS_DB_PASSWORD: 'YOUR_PASSWORD'
      WORDPRESS_DB_NAME: 'wd'
    volumes:
      - ./data/wordpress:/var/www/html
    networks:
      brg:
        ipv4_address: 192.168.234.2

  nginx:
    image: nginx:alpine
    container_name: nginx
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./tls_certs:/etc/sslcerts
      - ./data/wordpress:/var/www/html
    ports:
      - 80:80
      - 443:443
    restart: always
    networks:
      brg:
        ipv4_address: 192.168.234.3

  mysql:
    image: mysql:8.3.0-oraclelinux8
    container_name: mysql
    environment:
      MYSQL_DATABASE: 'wd'
      MYSQL_USER: 'wdpress'
      MYSQL_PASSWORD: 'YOUR_PASSWORD'
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - ./data/mysql:/var/lib/mysql
    restart: always
    networks:
      brg:
        ipv4_address: 192.168.234.4

networks:
  brg:
    ipam:
      driver: default
      config:
        - subnet: "192.168.234.0/24"
```

其中定义了3个容器和1个网络。需要注意的是，wordpress容器中的/var/www/html目录被映射到./data/wordpress，同时该目录被映射到nginx容器中。注意容器的tag，更新容器的时候需要选定同类型的版本。

### 配置nginx

推荐使用Mozilla SSL Configuration Generator来生成适合你的SSL配置。本文中的配置文件在其生成的配置文件基础上修改而来。

```nginx_conf
# generated 2024-02-26, Mozilla Guideline v5.7, nginx 1.17.7, OpenSSL 1.1.1k, intermediate configuration
# https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=intermediate&openssl=1.1.1k&guideline=5.7
server {
    listen 80;
    listen [::]:80;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name YOUR_SERVER_NAME;
    root /var/www/html;
    index index.php;

    ssl_certificate /etc/sslcerts/fullchain.pem;
    ssl_certificate_key /etc/sslcerts/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;
    ssl_session_tickets off;

    # curl https://ssl-config.mozilla.org/ffdhe2048.txt > /path/to/dhparam
    ssl_dhparam /etc/sslcerts/ffdhe2048.txt;

    # intermediate configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # replace with the IP address of your resolver
    resolver 223.5.5.5;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass 192.168.234.2:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

该配置文件已经实现了伪静态，无需更多配置。记得将下一节中配置的TLS证书写进配置文件中。

### 使用acme.sh配置TLS证书

acme.sh 实现了 acme 协议， 可以从 letsencrypt 生成免费的证书。

#### 安装acme.sh

使用官方的安装脚本安装acme.sh:

```bash
curl https://get.acme.sh | sh -s email=my@example.com
```

如果你的服务器在中国境内，可以使用gitee上的仓库安装acme.sh:

```bash
git clone https://gitee.com/neilpang/acme.sh.git
cd acme.sh
./acme.sh --install -m my@example.com
```

#### 申请并安装证书

推荐使用dns安装方式。参考<https://github.com/acmesh-official/acme.sh/wiki/dnsapi> 。
将证书安装到./tls_certs中，并更改相应名称。别忘了将你的域名指向你的站点，特别的，如果你的服务器位于中国大陆，要开启80和443端口需要ICP备案，请确认备案状态再进行测试。

### 启动

要启动配置好的docker compose，运行以下命令：

```bash
docker compose up
```

观察输出日志，直接使用域名访问服务器的443端口，进行初步配置。

如果要让服务以后台运行，可以添加参数：

```bash
docker compose up -d
```

容器就能后台运行并在服务器启动时自动启动。

## 小结

本部分中使用docker compose搭建了wordpress站点，并配置了TLS。下一节中将对其配置CDN访问以节省流量和防御攻击。
