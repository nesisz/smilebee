# openssl, nginx, php-fpm 설치및 서버 설정

1. openssl 설치

    ```
    wget https://www.openssl.org/source/openssl-1.0.2a.tar.gz
    tar -zxvf openssl-*.tar.gz
    cd openssl-*
    ./config -fpic shared && make && make install
    ```
2. php 설치

	```
	wget http://kr1.php.net/get/php-5.6.30.tar.gz/from/this/mirror
	```
	

	```
	./configure --prefix=/opt/php-5.6.30 --with-config-file-path=/opt/php-5.6.30/etc --with-mysql=/usr/local/mysql --with-mcrypt --with-gd --with-kerberos --enable-fpm --enable-sockets --enable-ftp --enable-inline-optimization --disable-debug --with-curl --with-zlib --with-iconv --with-jpeg-dir=/usr/lib --with-png-dir=/usr/lib --with-freetype-dir=/usr/lib --enable-mbstring --enable-mbregex --enable-sigchild --enable-dba --enable-mod_charset --enable-sysvsem --enable-soap --with-pdo-mysql --with-mysqli  --with-openssl=/usr/local/ssl --enable-maintainer-zts
	make && make install
	cp -f php.ini-production /opt/php-5.6.30/etc/php.ini
	cp -f /opt/php-5.6.30/etc/php-fpm.conf.default /opt/php-5.6.30/etc/php-fpm.conf

	mkdir -p /opt/php-5.6.30/logs
	ln -sf /dev/null /opt/php-5.6.30/logs/php-fpm.log

	cp -f ./sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm

	chmod +x /etc/init.d/php-fpm

	/sbin/chkconfig php-fpm on
    ```
3. nginx 설치

	1. 다운로드

	   ```
		wget http://nginx.org/download/nginx-1.10.3.tar.gz
		tar xvfz nginx-1.10.3.tar.gz
		cd nginx-1.10.3
	   ```
	2. 설치

		```
		./configure --prefix=/opt/nginx-1.10.3 \
	    --conf-path=/opt/nginx-1.10.3/conf/nginx.conf \
	    --sbin-path=/opt/nginx-1.10.3/sbin/nginx \
	    --lock-path=/var/lock/nginx.lock \
	    --pid-path=/var/run/nginx.pid \
	    --http-client-body-temp-path=/var/lib/nginx/body \
	    --http-proxy-temp-path=/var/lib/nginx/proxy \
	    --http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
	    --http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
	    --http-scgi-temp-path=/var/lib/nginx/scgi \
	    --http-log-path=/opt/nginx-1.10.3/log/access.log \
	    --error-log-path=/opt/nginx-1.10.3/log/error.log \
	    --with-http_addition_module \
	    --with-http_degradation_module \
	    --with-http_flv_module \
	    --with-http_gzip_static_module \
	    --with-http_image_filter_module \
	    --with-http_mp4_module \
	    --with-http_random_index_module \
	    --with-http_realip_module \
	    --with-http_ssl_module \
	    --with-http_stub_status_module \
	    --with-http_sub_module \
	    --with-http_realip_module \
	    --with-http_realip_module \
	    --user=nobody \
	    --group=nobody
	    
		make
		make install
		```
		```
	
		mkdir -p /var/lib/nginx
	
		ln -sf /dev/null /opt/nginx-1.10.3/log/access.log
		ln -sf /dev/null /opt/nginx-1.10.3/log/error.log
	
		vi /etc/init.d/nginx
	
		#!/bin/bash
		#
		# nginx - this script starts and stops the nginx daemin
		#
		# chkconfig:   - 85 15
		# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
	    #               proxy and IMAP/POP3 proxy server
	    # processname: nginx
	    # config:      /opt/nginx-1.10.3/conf/nginx.conf
	    # pidfile:     /var/run/nginx.pid
	
		# Source function library.
		. /etc/rc.d/init.d/functions
	
		# Source networking 		configuration.
		. /etc/sysconfig/network
	
		# Check that networking is up.
		[ "$NETWORKING" = "no" ] && exit 0
	
		nginx="/opt/nginx-1.10.3/sbin/nginx"
		prog=$(basename $nginx)
	
		NGINX_CONF_FILE="/opt/nginx-1.10.3/conf/nginx.conf"
	
		lockfile=/var/lock/subsys/nginx
	
		start() {
		    [ -x $nginx ] || exit 5
		    [ -f $NGINX_CONF_FILE ] || exit 6
		    echo -n $"Starting $prog: "
		    daemon $nginx -c $NGINX_CONF_FILE
		    retval=$?
		    echo
		    [ $retval -eq 0 ] && touch $lockfile
		    return $retval
		}
	
		stop() {
		    echo -n $"Stopping $prog: "
		    killproc $prog -QUIT
		    retval=$?
		    echo
		    [ $retval -eq 0 ] && rm -f $lockfile
		    return $retval
		}
	
		restart() {
		    configtest || return $?
		    stop
		    start
		}
	
		reload() {
		    configtest || return $?
		    echo -n $"Reloading $prog: "
		    killproc $nginx -HUP
		    RETVAL=$?
		    echo
		}
	
		force_reload() {
		    restart
		}
	
		configtest() {
		  $nginx -t -c $NGINX_CONF_FILE
		}
	
		rh_status() {
		    status $prog
		}
	
		rh_status_q() {
		    rh_status >/dev/null 2>&1
		}
	
		case "$1" in
		    start)
		        rh_status_q && exit 0
		        $1
		        ;;
		    stop)
		        rh_status_q || exit 0
		        $1
		        ;;
		    restart|configtest)
		        $1
		        ;;
		    reload)
		        rh_status_q || exit 7
		        $1
		        ;;
		    force-reload)
		        force_reload
		        ;;
		    status)
		        rh_status
		        ;;
		    condrestart|try-restart)
		        rh_status_q || exit 0
		            ;;
		    *)
		        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
		        exit 2
		esac
	
		chmod +x /etc/init.d/nginx
		/sbin/chkconfig nginx on
		```
	3. 설정

		```
		vi /opt/nginx-1.10.3/conf/nginx.conf
		```
		
		아래와같이 수정
		
		```
		server {
			listen       8080;
			server_name  localhost;
			
			root /mmrs_api;
			
			autoindex off;
			index index.php;
			
			location / {
			
			        try_files $uri $uri/ /index.php?/$request_uri;
			
			        location = /index.php {
			
			        fastcgi_pass unix:/var/run/php5-fpm.sock;
			        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
			        include        fastcgi_params;
			    }
			
			}
			
			location ~ \.php$ {
			        fastcgi_pass unix:/var/run/php5-fpm.sock;
			        fastcgi_index  index.php;
			        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
			        include        fastcgi_params;
			}
			error_page   500 502 503 504  /50x.html;
			location = /50x.html {
			    root   html;
			}
		}
		```
4. php-fpm 설정

	```
	vi /opt/php-5.6.30/etc/php-fpm.conf
	[global]
	pid = run/php-fpm.pid
	error_log = log/php-fpm.log

	[www]
	listen = /var/run/php5-fpm.sock
	listen.mode = 0666
	listen.allowed_clients = 127.0.0.1
	user = nobody
	group = nobody
	pm = dynamic
	pm.max_children = 50
	pm.start_servers = 20
	pm.min_spare_servers = 5
	pm.max_spare_servers = 35
	pm.max_requests = 500
    ```
5. 서버설정

	1. 추가된파일

		```
		/mmrs/server/server_add_cfg.api_euckr.php
		```
	2. 변경된파일

		```
		/mmrs/mrtg/thresh/thresh_in.php
		/mmrs/mrtg/thresh/thresh_out.php
		/mmrs/snmp/snmp_check.sh
		```
	3. DB 필드추가및 테이블 추가
		
		```
		## 앱로그인 세션 필드 추가
		ALTER TABLE `login_u` ADD `ses` VARCHAR(20)  NULL  DEFAULT NULL  AFTER `user`;
		
		## 푸시알림 받을 서버 필드 추가
		ALTER TABLE `server` ADD `push_st` VARCHAR(10)  NULL  DEFAULT NULL  AFTER `sms_st`;
		
		## 푸시내역 테이블
		CREATE TABLE `push_list` (
		  `No` int(11) unsigned NOT NULL AUTO_INCREMENT,
		  `msg` varchar(300) DEFAULT NULL,
		  `time` int(11) DEFAULT NULL,
		  `s_ip` varchar(20) DEFAULT NULL,
		  `type` varchar(50) DEFAULT NULL,
		  PRIMARY KEY (`No`),
		  KEY `s_ip` (`s_ip`),
		  KEY `type` (`type`)
		) ENGINE=MyISAM DEFAULT CHARSET=euckr;
		```
	4. crontab 에 등록
		
		```
		2,7,12,17,22,27,32,37,42,47,52,57 * * * * smileserv /opt/php-5.6.30/bin/php /mmrs_api/index.php crontab check port
		3,13,23,33,43,53 * * * * smileserv /opt/php-5.6.30/bin/php /mmrs_api/index.php crontab check index
		4,9,14,19,24,29,34,39,44,49,54,59 * * * * smileserv /opt/php-5.6.30/bin/php /mmrs_api/index.php crontab check snmp
		```
	5. 8080포트 방화벽해제

		```
		/etc/sysconfig/iptables
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
		```
