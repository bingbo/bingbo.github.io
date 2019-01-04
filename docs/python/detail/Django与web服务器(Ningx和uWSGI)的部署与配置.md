这里简单介绍django框架与nginx和uwsgi的部署与配置，可参考[官方文档](https://uwsgi.readthedocs.org/en/latest/tutorials/Django_and_nginx.html)

## 安装python

要用django框架当然得有安装python了，安装python如下所示：

```nginx
apt-get install python
```

或是

```nginx
brew install python
```

## 安装pip

可以参考[官网文档](https://pip.pypa.io/en/stable/)

1. 下载[get-pip.py](https://bootstrap.pypa.io/get-pip.py)

   ```nginx
   wget https://bootstrap.pypa.io/get-pip.py
   ```

2. 安装pip

   ```nginx
   python get-pip.py
   ```

## 安装django并生成项目

1. 直接用pip安装

   ```nginx
   pip install Django
   ```

2. 根据最新的源代码安装

   ```nginx
   git clone git://github.com/django/django.git
   pip install -e django/
   ```

3. 创建项目
   
   ```nginx
   django-admin.py startproject demo
   cd demo
   ```

## 安装uwsgi并运行

1. 执行下面命令安装uwsgi：

   ```nginx
   pip install uwsgi
   ```

2. 以文件形式配置uwsgi

   创建配置文件demo_uwsgi.ini，如下所示：

   ```nginx
   # mysite_uwsgi.ini file
   [uwsgi]

   # Django-related settings
   # the base directory (full path)
   chdir           = /Users/zhangbingbing/Public/www/demo
   # Django's wsgi file
   module          = demo.wsgi
   # the virtualenv (full path)
   #home            = /path/to/virtualenv

   # process-related settings
   # master
   master          = true
   # maximum number of worker processes
   processes       = 10
   # the socket (use the full path to be safe
   socket          = /Users/zhangbingbing/Public/www/demo/demo.sock
   # ... with appropriate permissions - may be needed
   # chmod-socket    = 664
   # clear environment on exit
   vacuum          = true
   ```

2. 启动uwsgi

   * 直接命令行启动

      ```nginx
      uwsgi --socket mysite.sock --module mysite.wsgi --chmod-socket=664
      ```

   * 以配置文件启动

      ```nginx
      uwsgi --ini demo_uwsgi.ini
      ```

## 配置nginx

1. 配置nginx，以使用请求发到uwsgi

   ```nginx
    ##8008端口for django frame
    server {
        listen       8003;
        server_name  localhost;
        charset      utf-8;

        client_max_body_size 75M;

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }


        location /static {
            root /Users/zhangbingbing/Public/www/demo;
        }
        location / {
            root /Users/zhangbingbing/Public/www/demo;
            uwsgi_pass      django;
            #uwsgi_pass unix:///Users/zhangbingbing/Public/www/demo/demo.sock;
            include         uwsgi_params;
            #uwsgi_read_timeout 180;
        }

        location ~ /\.ht {
            deny  all;
        }
    }

    upstream django{
        #server 127.0.0.1:9999;
        server unix:///Users/zhangbingbing/Public/www/demo/demo.sock;
    }
   ```

2. 启动nginx

   ```nginx
   nginx -s stop
   nginx -c /usr/local/etc/nginx/nginx.conf
   nginx -s reload
   ```