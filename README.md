# nginx-ubuntu
1. Chuẩn bị các thông số ban đầu
[root@athena ~]# vi /etc/sysconfig/selinux
SELINUX=disabled
[root@athena ~]# cat /etc/hostname
athena.local
[root@athena ~]# cd /etc/sysconfig/network-scripts/
[root@athena network-scripts]# ls
ifcfg-ens32  ifdown-ippp  ifdown-routes    ifup          ifup-ipv6   ifup-ppp       ifup-tunnel
ifcfg-lo     ifdown-ipv6  ifdown-sit       ifup-aliases  ifup-isdn   ifup-routes    ifup-wireless
[root@athena network-scripts]# vi ifcfg-ens32
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens32
DEVICE=ens32
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.1.178
NETMASK=255.255.255.0
GATEWAY=192.168.1.254
DNS1=8.8.8.8
[root@athena ~]# systemctl stop firewalld
[root@athena ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
2. Download và cài đặt Nginx từ mã nguồn
[root@athena ~]# dnf groupinstall -y 'Development Tools' && dnf install -y vim
[root@athena ~]# dnf install -y epel-release iptables-services
[root@athena ~]# dnf install -y perl perl-devel perl-ExtUtils-Embed libxslt libxslt-devel libxml2 libxml2-devel gd gd-devel GeoIP GeoIP-devel
[root@athena ~]# wget http://nginx.org/download/nginx-1.20.1.tar.gz
[root@athena ~]# wget https://ftp.pcre.org/pub/pcre/pcre-8.45.tar.gz
[root@athena ~]# wget https://www.zlib.net/zlib-1.2.11.tar.gz
[root@athena ~]# wget https://www.openssl.org/source/openssl-1.1.1k.tar.gz
[root@athena ~]# systemctl stop iptables
[root@athena ~]# tar xf pcre-8.45.tar.gz
[root@athena ~]# tar xf zlib-1.2.11.tar.gz
[root@athena ~]# tar xf openssl-1.1.1k.tar.gz
[root@athena ~]# tar xf nginx-1.20.1.tar.gz
[root@athena ~]# cd nginx-1.20.1
[root@localhost nginx-1.20.1]# cp man/nginx.8 /usr/share/man/man8
[root@localhost nginx-1.20.1]# gzip /usr/share/man/man8/nginx.8
[root@localhost nginx-1.20.1]# man nginx
[root@athena nginx-1.20.1]# ./configure --prefix=/etc/nginx \
            --sbin-path=/usr/sbin/nginx \
            --modules-path=/usr/lib64/nginx/modules \
            --conf-path=/etc/nginx/nginx.conf \
            --error-log-path=/var/log/nginx/error.log \
            --pid-path=/var/run/nginx.pid \
            --lock-path=/var/run/nginx.lock \
            --user=nginx \
            --group=nginx \
            --build=Rocky-Linux \
            --builddir=nginx-1.20.1 \
            --with-select_module \
            --with-poll_module \
            --with-threads \
            --with-file-aio \
            --with-http_ssl_module \
            --with-http_v2_module \
            --with-http_realip_module \
            --with-http_addition_module \
            --with-http_xslt_module=dynamic \
            --with-http_image_filter_module=dynamic \
            --with-http_geoip_module=dynamic \
            --with-http_sub_module \
            --with-http_dav_module \
            --with-http_flv_module \
            --with-http_mp4_module \
            --with-http_gunzip_module \
            --with-http_gzip_static_module \
            --with-http_auth_request_module \
            --with-http_random_index_module \
            --with-http_secure_link_module \
            --with-http_degradation_module \
            --with-http_slice_module \
            --with-http_stub_status_module \
            --http-log-path=/var/log/nginx/access.log \
            --http-client-body-temp-path=/var/cache/nginx/client_temp \
            --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
            --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
            --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
            --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
            --with-mail=dynamic \
            --with-mail_ssl_module \
            --with-stream=dynamic \
            --with-stream_ssl_module \
            --with-stream_realip_module \
            --with-stream_geoip_module=dynamic \
            --with-stream_ssl_preread_module \
            --with-compat \
            --with-pcre=../pcre-8.45 \
            --with-pcre-jit \
            --with-zlib=../zlib-1.2.11 \
            --with-openssl=../openssl-1.1.1k \
            --with-openssl-opt=no-nextprotoneg \
            --with-debug
[root@localhost nginx-1.20.1]# make
[root@localhost nginx-1.20.1]# make install
[root@athena ~]# ln -s /usr/lib64/nginx/modules /etc/nginx/modules
[root@athena ~]# cd  /etc/nginx/modules
[root@athena modules]# ls
ngx_http_geoip_module.so         ngx_http_xslt_filter_module.so  ngx_stream_geoip_module.so
ngx_http_image_filter_module.so  ngx_mail_module.so              ngx_stream_module.so
[root@athena ~]# useradd --system --home /var/cache/nginx --shell /sbin/nologin --comment "nginx user" --user-group nginx
[root@athena ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (2: No such file or directory)
nginx: configuration file /etc/nginx/nginx.conf test failed
[root@athena ~]# mkdir -p /var/cache/nginx
[root@athena ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
3. Tạo scripts để khởi động máy chủ Nginx
[root@athena ~]# vim /usr/lib/systemd/system/nginx.service
[Unit]
Description=nginx - high performance web server
Documentation=https://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target
[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
[Install]
WantedBy=multi-user.target
[root@athena ~]# systemctl start nginx
[root@athena ~]# systemctl enable nginx
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.
