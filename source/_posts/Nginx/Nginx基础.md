---
title: Nginx基础
date: 2018-11-09 16:13:08
tags: Nginx
categories: 技术
---

### Nginx特征

- IO多路复用epoll

    - 多个描述符的I/O操作都是在一个线程内并发交替地顺序完成。这里的”复用“指的是复用同一个线程。
    - epoll模型：相较select模型更加高效。每当FD就绪，采用系统的回调函数之间fd放入，效率更高；最大 连接数无限制。Nginx就是采用了epoll模型

- 轻量级

    - 功能模块少
    - 代码模块化

- CPU亲和（affinity）

    <!--more-->

    是一种把CPU核心和Nginx工作进程绑定方式，把每个worker进程固定到一个cpu上执行，减少切换cpu的cache miss，获得更好的性能。

- sendFile

    ![](https://ws4.sinaimg.cn/large/006tNbRwgy1fx1w4ghww8j30hp0ayn31.jpg)

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fx1w5jok7jj30i60bijwu.jpg)

`rpm -ql nginx` 查看nginx的目录(linux rpm安装方式)

```
/etc/logrotate.d/nginx
/etc/nginx/fastcgi.conf
/etc/nginx/fastcgi.conf.default
/etc/nginx/fastcgi_params
/etc/nginx/fastcgi_params.default
/etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/mime.types
/etc/nginx/mime.types.default
/etc/nginx/nginx.conf
/etc/nginx/nginx.conf.default
/etc/nginx/scgi_params
/etc/nginx/scgi_params.default
/etc/nginx/uwsgi_params
/etc/nginx/uwsgi_params.default
/etc/nginx/win-utf
/usr/bin/nginx-upgrade
/usr/lib/systemd/system/nginx.service
/usr/lib64/nginx/modules
/usr/sbin/nginx
/usr/share/doc/nginx-1.12.2
/usr/share/doc/nginx-1.12.2/CHANGES
/usr/share/doc/nginx-1.12.2/README
/usr/share/doc/nginx-1.12.2/README.dynamic
/usr/share/doc/nginx-1.12.2/UPGRADE-NOTES-1.6-to-1.10
/usr/share/licenses/nginx-1.12.2
/usr/share/licenses/nginx-1.12.2/LICENSE
/usr/share/man/man3/nginx.3pm.gz
/usr/share/man/man8/nginx-upgrade.8.gz
/usr/share/man/man8/nginx.8.gz
/usr/share/nginx/html/404.html
/usr/share/nginx/html/50x.html
/usr/share/nginx/html/index.html
/usr/share/nginx/html/nginx-logo.png
/usr/share/nginx/html/poweredby.png
/usr/share/vim/vimfiles/ftdetect/nginx.vim
/usr/share/vim/vimfiles/indent/nginx.vim
/usr/share/vim/vimfiles/syntax/nginx.vim
/var/lib/nginx
/var/lib/nginx/tmp
/var/log/nginx
```

或者通过`nginx -V`查看相应参数



### Nginx的默认配置





