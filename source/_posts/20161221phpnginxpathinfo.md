---
title: php pathinfo 以php为后缀的时候nginx配置修改
date: 2016-12-21 22:40:16
tags:
- Apache
- Pathinfo
- Nginx
categories:
- Nginx
---
这个是一个apache的pathinfo的配置，pathinfo为/verify_login.html或者为/verify_login.php是可以访问的
```
# BEGIN Speedy
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php?$1 [L]
</IfModule>
# END Speedy
```
这个是个nginx的配置文件,当pathinfo为/verify_login.php时提示文件不存在，转发不到入口文件index.php上
```
 location ~ [^/]\.php(/|$)
        {
            fastcgi_pass  unix:/tmp/php-cgi.sock;
            fastcgi_index index.php;
            include fastcgi.conf;
            include pathinfo.conf;
        }
  #fastdfs的配置文件
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```
当这个php文件不存在的时候，需要修改配置文件SCRIPT_FILENAME为index.php，将verify_login.php当做QUERY_STRING
修改后的fastdfs的配置文件为：
```
if (!-e $request_filename) {
     set $m $fastcgi_script_name;
     set $n "/index.php";
     set $w $request_uri;
     set $q $query_string;
}

if (-e $request_filename) {
     set $m $fastcgi_script_name;
     set $n $fastcgi_script_name;
     set $w $request_uri;
     set $q $query_string;
}


fastcgi_param  SCRIPT_FILENAME    $document_root$n;
fastcgi_param  QUERY_STRING       $q;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $n;
fastcgi_param  REQUEST_URI        $w;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

#PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```
判断文件是否存在，假如存在 SCRIPT_FILENAME 为/verify_login.php
假设不存在， SCRIPT_FILENAME 为/index.php  将verify_login.php，实际请求地址变为/index.php/verify_login.php