### 主配置文件nginx.conf

```nginx


user  nobody;
####work进程数，可以设置为cpu数据，auto将自动检测
worker_processes    4;
worker_cpu_affinity 0001 0010 0100 1000;

#worker_processes auto;
#worker_cpu_affinity auto;

worker_rlimit_nofile 100000;

error_log  logs/error.log   warn;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

pid        logs/nginx.pid;


events {
    ###同时打开的最大连接数
    worker_connections  1024;
    ###收到一个新连接后接受尽可能多的连接
    multi_accept on;
    use epoll;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

#    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
#                      '$status $body_bytes_sent "$http_referer" '
#                      '"$http_user_agent" "$http_x_forwarded_for" "$request_time" "$upstream_response_time"';

    ###设置nginx是否将存储访问日志。关闭这个选项可以让读取磁盘IO操作更快(aka,YOLO)
    #access_log  logs/access.log  main;
    ###不发送服务器版本
    server_tokens off;
    ###打开，直接从一个地方cp到另一文件，不在放缓存
    sendfile        on;
    ###启用tcp_nopush选项与sendfile on结合，可以使nginx在通过sendfile获取数据后发送http响应头在一个包中,在一个数据包里发送所有头文
    tcp_nopush     on;
    ###关闭Nagle算法，直接发送忽略包的大小或等待到达一定大小的包，不要缓存数据，及时发送
    tcp_nodelay       on;
    ###连接超时时间
    keepalive_timeout  65;
    ###设置读取客户端请求体的缓存大小
    client_body_buffer_size 10K;
    ###设置读取客户端请求头的缓存大小
    client_header_buffer_size 1k;
    ###设置允许客户端发送请求体的最大大小，在Content-Length里指定的，超过该值则返回413(Request Entity Too Large)错误
    client_max_body_size 8m;
    ###设置读取客户端请求头的缓存的最大数量及大小num,size
    large_client_header_buffers 2 1k;
    ###设置请求头或请求体的超时时间
    client_header_timeout 10;
    client_body_timeout 10; 
    ###关闭不响应的客户端连接，释放内存空间
    reset_timedout_connection on;
    ###响应超时时间
    send_timeout 10s;
    ###用于保存各种key（比如当前连接数）的共享内存的参数
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    ###为给定的key设置最大连接数
    limit_conn addr 100;
    ###打开缓存并设置缓存最大数目，缓存时间，在不活动时间超过后清除
    open_file_cache max=100000 inactive=20s;
    ###在open_file_cache中指定检测正确信息的间隔时间
    open_file_cache_valid 30s; 
    ###定义了open_file_cache中指令参数不活动时间期间里最小的文件数
    open_file_cache_min_uses 2; 
    ###指定了当搜索一个文件时是否缓存错误信息
    open_file_cache_errors on;
    ###开启压缩
    gzip  on;
    ###最小压缩大小
    gzip_min_length 1000;
    ###允许或者禁止压缩代理请求的响应。我们设置为any，意味着将会压缩所有的请求
    gzip_proxied any;
    ###设置数据的压缩级别
    gzip_comp_level 4;
    ###设置需要压缩的数据格式
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    ###默认字符集
    charset UTF-8;

    server {
        listen       8900;
        server_name  localhost;


        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        location ~* .(woff|eot|ttf|svg|mp4|webm|jpg|jpeg|png|gif|ico|css|js)$ {
            expires 365d;
        }

    }

    include vhost/*.conf;
    

}

```

### 应用服务配置vhost/demo.conf

```nginx
server {
    listen       8101;
    server_name  localhost;


    root /home/work/html/web;
    location / {
        index  index.html index.htm index.php;
        try_files $uri $uri/ /index.php?$query_string;
    }


    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

    location ~* \.(?:css|js|png|jpg)$ {
        expires 1y;
        access_log off;
        add_header Cache-Control "public";
    }

    
    location ~ \.php$ {
        add_header Access-Control-Allow-Origin *; 
        fastcgi_pass    unix:/home/work/php5/var/run/php-fpm.sock;
        #fastcgi_pass    unix:/home/work/php7/var/run/php-fpm.sock;
        #fastcgi_pass   127.0.0.1:9001;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    location ~ /\.ht {
        deny  all;
    }

}
```