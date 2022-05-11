# 编译安装 Nginx
 
## 安装 GCC 与 dev 库
- GCC编译器 `yum -y install gcc gcc-c++`
- 正则表达式PCRE库 `yum install -y pcre pcre-devel`
- zlib压缩库 `yum install -y zlib zlib-devel`

- OpenSSL开发库: 为了指定版本，使用源码安装 
`wget https://www.openssl.org/source/openssl-1.1.1n.tar.gz`
`tar -zxvf openssl-1.1.1n.tar.gz`
`cd openssl-1.1.1n`
`mkdir /usr/local/openssl`
`./config --prefix=/usr/local/openssl`
`./config -t`
`make`
`make install`
添加系统配置
创建文件 `vim /etc/ld.so.conf.d/openssl.conf`
填入内容 `/usr/local/openssl`
加载配置：ldconfig

## 下载并解压nginx
```shell
wget https://nginx.org/download/nginx-1.21.6.tar.gz`
tar -zxvf nginx-1.21.6.tar.gz
cd nginx-1.21.6
```

## 安装目录规划
- nginx安装目录 `/usr/local/nginx`
- nginx配置文件目录 `/usr/local/nginx/nginx.conf`
- nginx虚拟服务器配置目录 `/usr/local/nginx/vhost/`
- log日志目录：`/var/log/nginx`
- pid文件目录：`/var/run/nginx.pid`
- lock锁目录：`/var/run/nginx.lock`
- 临时缓存目录：`/var/cache/nginx`
- 站点目录：`/www/wwwroot/`
- nginx运行用户名：`nginx`
- nginx运行用户组：`nginx`

## configure的命令参数
- 列出configure包含的参数： `./configure --help`

**通用配置项**
|选项|解释|
|----|-----|
|--prefix=PATH |set installation prefix|
|--sbin-path=PATH|set nginx binary pathname|
|--conf-path=PATH|set nginx.conf pathname|
|--error-log-path=PATH|set error log pathname|
|--pid-path=PATH|set nginx.pid pathname|
|--lock-path=PATH|set nginx.lock pathname|
|--user=USER|set non-privileged user for worker processes|
|--group=GROUP|set non-privileged user for worker processes|
|--with-file-aio|enable file AIO support 为FreeBSD 4.3+和linux 2.6.22+系统启用异步I/O|
| --with-debug|enable debug logging 生产环境中不推荐使用|

**临时路径配置选项**
|选项|解释|
|----|-----|
|--error-log-path=PATH |set error log pathname|
|--http-log-path=PATH |set http access log pathname http 访问日志的默认路径|
|--http-client-body-temp-path=PATH |set path to store http client request body temporary files 从客户端收到请求后，该选项设置的目录用于作为请求体 临时存放的目录。如果 WebDAV 模块启用，那么推荐设置 该路径为同 一文件系统上的目录作为最终的目的地|
|--http-proxy-temp-path=PATH|set path to store http proxy temporary files 在使用代理后，通过该选项设置存放临时文件路径 |
|--http-fastcgi-temp-path=PATH|set path to store http fastcgi temporary files 设置 FastCGI 临时文件的目录|
|--http-uwsgi-temp-path=PATH|set path to store http uwsgi temporary files|
|--http-scgi-temp-path=PATH|set path to store http scgi temporary files|

**PCRE的配置参数**
|选项|解释|
|----|-----|
|--without-pcre|disable PCRE library usage 如果确定Nginx不用解析正则表达式，那么可以使用这个参数|
|--with-pcre|force PCRE library usage 强制使用PCRE库|
|--with-pcre=DIR|set path to PCRE library sources|
| --with-pcre-opt=OPTIONS|set additional build options for PCRE 编译PCRE源码时希望加入的编译选项|

**OpenSSL的配置参数**
|选项|解释|
|----|-----|
|--with-openssl=DIR|set path to OpenSSL library sources 指定OpenSSL库的源码位置，在编译nginx时会进入该目录编译OpenSSL，如果web服务器需要使用HTTPS，那么Nginx要求必须使用OpenSSL|
|--with-openssl-opt=OPTIONS|set additional build options for OpenSSL 编译OpenSSL源码时希望加入的编译选项|

**zlib的配置参数**
|选项|解释|
|----|-----|
|--with-zlib=DIR|set path to zlib library sources 指定zlib库源码的位置，在编译nginx时会进入该目录编译zlib，如果需要使用gzip压缩就必须要zlib库支持|
|--with-zlib-opt=OPTIONS|set additional build options for zlib 编译zlib源码时希望加入的编译选项|
|--with-zlib-asm=CPU|use zlib assembler sources optimized for the specified CPU, valid values: pentium, pentiumpro 指定对特定CPU使用zlib库的汇编优化功能，目前支持两种架构：petium和pentiumpro|

其他通过`./configure --help` 查看

## Nginx编译步骤
创建nginx用户和用户组
```
groupadd nginx
useradd -g nginx nginx
```
进入nginx源码包
生成makefile文件
```
./configure \
--prefix=/usr/local/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--user=nginx \
--group=nginx \
--with-file-aio \
--with-threads \
--with-http_addition_module \
--with-http_auth_request_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_mp4_module \
--with-http_random_index_module \
--with-http_realip_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_sub_module \
--with-http_v2_module \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-stream_realip_module \
--with-stream_ssl_module \
--with-stream_ssl_preread_module \
--with-openssl=/root/openssl-1.1.1n
```
编译安装
```
make && make install
# 安装成功执行以下命令查看nginx版本号
[root@localhost nginx-1.21.6]# nginx -v
nginx version: nginx/1.21.6
```

## 使用自签ssl证书

**CA根证书的生成步骤**

生成CA私钥（.key）-->生成CA证书请求（.csr）-->自签名得到根证书（.crt）（CA给自已颁发的证书）

```shell
# Generate CA private key 
openssl genrsa -out ca.key 2048 
# Generate CSR 
openssl req -new -key ca.key -out ca.csr
# Generate Self Signed certificate（CA 根证书）
openssl x509 -req -days 3650 -in ca.csr -signkey ca.key -out ca.crt
```

自己做了的免费证书的网站不能直接使用 curl访问
可以把crt和key文件放在一起

`cat ca.crt ca.key >> ca.pem`
`curl --cacert ca.pem https://erniu.fun -I`

## 站点配置

```nginx
server {
    listen  443 ssl;
    server_name erniu.fun;
    ssl_certificate certs/ca.crt;
    ssl_certificate_key certs/ca.key;

    location / {
        root /www/wwwroot/erniufun;
        index index.html index.htm;
    }
}
```

