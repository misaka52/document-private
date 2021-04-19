## 基础

### 1. MAC安装

Brew info nginx  查看是否安装nginx，若未安装直接brew install nginx 安装软件

安装nginx后信息如下

```
➜  nginx brew info nginx
nginx: stable 1.19.6 (bottled), HEAD
HTTP(S) server and reverse proxy, and IMAP/POP3 proxy server
https://nginx.org/
/usr/local/Cellar/nginx/1.19.6 (25 files, 2.2MB) *
  Poured from bottle on 2021-04-18 at 00:23:52
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/nginx.rb
License: BSD-2-Clause
==> Dependencies
Required: openssl@1.1 ✔, pcre ✔
==> Options
--HEAD
	Install HEAD version
==> Caveats
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
==> Analytics
install: 49,834 (30 days), 137,515 (90 days), 487,989 (365 days)
install-on-request: 49,707 (30 days), 137,195 (90 days), 481,494 (365 days)
build-error: 0 (30 days)
```

启动

```nginx
# 校验nginx配置文件正确性，一般启动都要先手动校验一下
nginx -t
# 启动nginx
nginx
# 关闭nginx
nginx -s stop/quit
# 重启nginx
nginx -s reload/reopen
```

