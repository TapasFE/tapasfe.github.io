---
title: TapasTech 使用 Let's Encrypt 证书流程
date: 2016-09-14 22:37:08
tags:
  - tech
  - ssl
  - https
  - cert
---

[Let's Encrypt](https://letsencrypt.org/) 的强大之处就不多说了。由于种种原因，我们决定迁移到 Let's Encrypt 证书，而且其可以一次性包含多个域名，非常方便。

## 准备工作
1. 请自行补充 HTTPS 相关知识。

1. Let's Encrypt 证书的签发是免费的，但是需要通过网络来认证服务器的身份，所以签发过程有两个必要条件：

   1. 服务器可以连接到 Let's Encrypt ；
   1. Let's Encrypt 可以连接到我们的服务器。

   因此，证书的签发过程必须在一台拥有公网 IP 的服务器上，也就是有对外的 NginX 的服务器上，而且最终我们的证书文件也是要让 NginX 直接访问的。

1. 偷懒是程序员的美德，这里使用[我的脚本集](https://github.com/gera2ld/scripts/tree/master/letsencrypt)，clone 下来就好了。

## 签发证书
1. （仅一次）运行脚本1，生成账号密钥。
1. （仅一次）运行脚本2，下载[acme-tiny](https://github.com/diafygi/acme-tiny)脚本。
1. （每次域名修改后）运行脚本3，生成 CSR（Certificate Signing Request）。

   域名的修改已经抽出来放到 `/root/letsencrypt/domains.txt` 了。

1. 配置 NginX 的 HTTP 部分，以达到如下目的：

   1. 提供 `challenges` 文件夹静态服务，用于 Let's Encrypt 签发证书时验证身份。
   1. 自动将 HTTP 请求跳转到相应的 HTTPS 地址访问。

   具体配置如下：

   ```
   server {
       listen 80;
       server_name a.dtcj.com b.dtcj.com c.dtcj.com;

       location /.well-known/acme-challenge/ {
           alias /absolute/path/to/challenges/;
           try_files $uri =404;
       }

       location / {
           return 301 https://$host$request_uri;
       }
   }
   ```

1. （定期）运行脚本4，到 Let's Encrypt 签发证书。

   值得注意的是，证书有效期为 90 天，所以需要定期刷新证书，重复执行此脚本即可。一个比较好的方法是将此脚本加入 crontab ：

   ```
   0 0 1 * * /absolute/path/to/4.cert.sh >/dev/null 2>&1
   ```

至此，证书签发成功，再配置 NginX 的 HTTPS 部分如下：
```
server {
    listen       443 ssl;
    server_name  a.dtcj.com b.dtcj.com c.dtcj.com;

    ssl_certificate      /absolute/path/to/chained.pem;
    ssl_certificate_key  /absolute/path/to/domain.key;

    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers   on;

    location / {
        proxy_pass       http://localhost:3000;
        proxy_set_header Host      $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        alias /absolute/path/to/static;
        try_files $uri =404;
    }
}
```
