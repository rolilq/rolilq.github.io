---
layout:     post
title:      使用Let's Encrypt 给Ubuntu服务器配置https
subtitle:   配置https
date:       2020-3-12
author:     AHui
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - https
---
#### 使用Cerbot 申请证书

下载Cerbot

```
git clone https://github.com/certbot/certbot
cd cerbot
```

-d后边跟域名，dns01是给域名添加一个DNS TXT记录

```
./certbot-auto certonly --manual -d yyxw.ciisw.com --manual-public-ip-logging-ok --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory
```

当输入完上述命令之后，填写确认选项之后，输入完邮箱，需要验证域名所有权

```
Please deploy a DNS TXT record under the name
_acme-challenge.example.com with the following value:

mhumL1xJOHPIZtFTEm4rotjJnR9TdkBVPuCS9YHvNjs

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

此时先不按回车，去阿里云域名管理添加解析。

记录类型TXT

主机记录填上面的**_acme-challenge.example.com**

记录值输入上面生成的随机字符串**mhumL1xJOHPIZtFTEm4rotjJnR9TdkBVPuCS9YHvNjs** 

然后在另一台linux系统上运行命令

```
dig -t txt _acme-challenge.example.com @8.8.8.8
```

正确的话会出现,会有两个配置文件的位置在nginx的配置中会用到

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/yyxw.ciisw.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/yyxw.ciisw.com/privkey.pem
   Your cert will expire on 2020-05-02. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

#### nginx配置https强制开启https

安装nginx：https://blog.csdn.net/qq_28242451/article/details/103821884

没有ssl模块的解决方法：https://blog.seosiwei.com/detail/28

重载nginx的配置 : **/usr/local/nginx/sbin/nginx -s reload**

nginx配置文件

监听443端口 写 listen 443 ssl，如果写ssl on 会发出警告，表示这个写法已经过时

```nginx
 server {
	listen       80;
    server_name  yyxw.ciisw.com;
    return 301 https://$server_name$request_uri;
}
server {
    listen       443 ssl;
    server_name  yyxw.ciisw.com;
    #  ssl on;

    ssl_certificate      /etc/letsencrypt/live/yyxw.ciisw.com/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/yyxw.ciisw.com/privkey.pem;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        proxy_pass http://127.0.0.1:8088;
    }
}
```
