
学习nginx后，结合相应的文档，对其进行相应的总结分享。





## nginx的基本功能
### 配置虚拟服务器 
> nginx配置文件必须包含至少一个`server`指令来定义一个虚拟主机。当nginx处理一个请求时，他首先会选择一个虚拟主机来服务该请求。

> 一个虚拟主机通过一个`server`指令被定义到`http`上下文中，例如：

```nginx
http {
    server {
        # Server configuration
        listen 127.0.0.1:8080 default_server;
        server_name example.org www.example.org;
    }
}
```
> 也可以添加多个`server`指令在`http`上下文中来定义多个虚拟服务器。

> 虚拟服务器块通常包含一个`listen`指令一个IP地址和端口号使其来监听请求。
如果端口号省略，则标准的端口会被使用。同理如果IP地址活力了，则服务器会监听所有的IP地址。
如果省略了`listen`指令，则标准端口为80和默认端口为8080，这主要依赖于用户的权限。

> 如果有多个服务匹配了请求的相应的IP和端口，nginx会测试请求的头`Host`字段是否与`server_name`指令的值匹配。`server_name`的参数可以是全名、模糊名或是正则表达式。如果匹配，则nginx会按如下顺序查找并使用第一个匹配的项：
  
1. 准确匹配的名字  
2. 最长的以\*通配符开头的，如*.xxx.org  
3. 最条的以\*通配符结束的，如mail.*  
4. 第一个匹配的正则表达式 

> 如果`Host`字段没有匹配的服务，nginx把请求路由到端口相应的默认的服务上。  
默认的服务一般是在**nginx.conf**文件中的第一个服务，除非你用`default_server`指令指定一个作默认服务。 

### 配置Locations 
> nginx可以发送网络到不同的代理上或服务于不同的文件基于请求的URLs，这些块使用`location`指令将其定义在`server`指令块里。

> nginx会测试请求的URLS匹配所有`location`指令的参数并执行所有的指令在匹配的`location`上。在每个`location`块内，通常可能放置多个`location`指令去进一步细化处理指定的一组请求。

> 这里有两种类型的`location`指令参数：*前缀字符串*和*正则表达式*。一个请求URL要匹配一个前缀字符串，它必须是以该字符串开头。

```nginx
location /some/path/ {
    ...
}
location ~ \.html? {
    ...
}

location /images/ {
    root /data;
}
location / {
    proxy_pass http://www.example.com;
}
```
> 上面是路径前缀匹配`location`和正则表达式匹配`location`，   
~正则表达式是大小写敏感匹配，   
~* 是大小写不敏感匹配。      
为了能更好的匹配一个URL，nginx首先用url与前缀字符串匹配的locations比较，之后再去查找带有正则表达式的locations     
选择location处理请求的准确逻辑如下所示：   

1. 测试URL与所有的前缀字符串   
2. = 修饰符定义一个准确的URL匹配和前缀字符串匹配，如果找到了精确匹配，则停止查找。   
3. ^~ 修饰符优化考虑最长前缀匹配字符串，不会对正则表达式作检查   
4. 存储最长匹配前缀字符串   
5. 测试URI是否匹配正则表达式   
6. 在第一个匹配的正则表达式处停止并使用相应的location   
7. 如果没有相应的正则表达式匹配，则使用存储的相应的前缀字符串匹配   

> 一个location上下文可以包含多个指令去定义如果解决一个请求或服务一个静态文件或发送请求给一个代理服务器。   
`root`指令指定一个文件路径根目录，在这里查找请求的静态文件。请求的URL拼接在该目录后面组成静态文件的全路径名。   
`proxy_pass`指令转发请求到代理服务器，返回的响应给传回给客户端。

### 使用变量 
> 你可以在确定的情况下，在配置文件中使用变量来处理请求。一个变量以$符号作为开头，可以使用`set`,`map`,`geo`等指令来定义变量，大多数变量在运行时定义，包含相应请求的一些信息，例如core HTTP模块中的变量：

* `$arg_name` 请求行中的参数名
* `$args` 请求行中的参数
* `$binary_remote_addr` 客户端地址，值的长度总是4bytes
* `$body_bytes_sent` 发送给客户端的字节数，不算响应头
* `$bytes_sent` 发送给客户端的字节数
* `$connection` 连接序列号
* `$connection_requests` 当前已连接的请求数
* `$content_length` 请求头的Content-Length字段
* `$content_type` 请求头的Content-Type
* `$cookie_name` cookie名
* `$document_root` 当前请求的`root`和`alias`指令的值
* `$document_uri` 和`$uri`一样
* `$host` 请求行的host name或是请求头的Host或是匹配一个请求的server name
* `$hostname` host name
* `$http_name` 任意请求头字段，
* `$https` 如果连接操作是在SSL模式下为'on'，否则为空
* `$is_args` 如果请求行有参数则为'?'，否则为空
* `$limit_rate` 响应速度限制
* `$msec` 当前时间毫秒
* `$nginx_version` nginx版本
* `$pid` 工作进程的pid
* `$pipe` 如果是流水线请求则为“p”，否则为空
* `$proxy_protocol_addr` 来自代理协议头的客户端地址
* `$query_string` 和`$args`一样，所有的请求参数
* `$realpath_root` 绝对路径
* `$remote_addr` 客户端地址
* `$remote_port` 客户端端口
* `$remote_user` 基本验证用的用户名
* `$request` 全部请求行信息
* `$request_body` 请求体
* `$request_body_file` 请求体的临时文件名
* `$request_completion` 如果请求完成了则为“OK”，否则为空
* `$request_filename` 当前请求的文件路径，基于`root`和`alias`指令参数
* `$request_length` 请求长度，包含（请求行、请求头部、请求体）
* `$request_method` 请求方法，“GET” or “POST”
* `$request_time` 请求时间
* `$request_uri` 全部原请求URI（包含参数）
* `$scheme` 请求scheme,"http" or "https"
* `$sent_http_name` 所有的相应头字段
* `$server_addr` 接收请求的服务器地址
* `$server_name` 接收请求的服务器名
* `$server_port`接收请求的服务器端口
* `$server_protocol` 请求协议，一般为“HTTP/1.0”, “HTTP/1.1”, or “HTTP/2.0”
* `$status 响应状态码
* `$tcpinfo_rtt, $tcpinfo_rttvar, $tcpinfo_snd_cwnd, $tcpinfo_rcv_space` 客户端TCP连接的信息
* `$time_iso8601` ISO标准格式的本地时间
* `$time_local` 通用日志格式的本地时间
* `$uri` 当前请求的URI

### 返回特定的状态码
> 一些网站URLs需要立即返回带指定错误或重定向的状态码的响应，比如当页面被临时或永久移除时。要达到这种效果最简单的方式就是使用`return`指令，例如：

```nginx
location /wrong/url {
    return 404;
}
location /permanently/moved/url {
    return 301 http://www.example.com/moved/here;
}
```
> `return`指令的第一个参数是响应状态码。第二个可选的参数可以是一个重定向的URL或响应体中的文本内容。  
`return`指令可以放在`location`和`server`上下文中。

### 重写请求的URLs
> 一个请求URL可以使用`rewrite`指令被多次修改在请求处理过程中，其中包含一个可选的和两个必须的参数。第一个必须的参数为请求URL必须匹配的正则表达式。第二个参数是需要替换匹配URI的URI。第三个可选的参数是一个标识，用来停止重写的继续处理或发送一个重定向。例如：

```nginx
location /users/ {
    rewrite ^/users/(.*)$ /show?user=$1 break;
}
```
> 你可以包含多个`rewrite`指令在server和location上下文中，nginx按顺序一个一个的执行相应的指令。在server上下文中的`rewrite`指令仅执行一次当被选中的时候。

> 这里有两个可以中断执行`rewrite`指令的参数：

* last - 停止执行rewrite指令在当前的server或location里，但nginx会继续查找与重写后的URI匹配的location,任何在新的location里的rewrite指令都会执行
* break - 停止处理rewrite指令在当前的context中，同时取消查询与新URI匹配的location,在新的location中的rewrite指令也不会执行

### 重写HTTP Responses
> 有时你需要重写或改变一个HTTP响应体中的内容，替换一个字符串为另一个。您可以使用sub_filter指令来定义重写应用。该指令支付变量和替换链，做更复杂的改变。

```nginx
location / {
    sub_filter     'href="http://127.0.0.1:8080/'    'href="http://$host/';
    sub_filter     'img src="http://127.0.0.1:8080/' 'img src="http://$host/';
    sub_filter_once on;
}
```
而`sub_filter_once`指定则告诉nginx在location中连续应用`sub_filter`指令。需要注意的是已经被`sub_filter`指令修改的响应内容不会被再次替换，如果另一个sub_filter匹配发生时。

### 处理错误
> 用`error_page`指令你可以配置nginx返回一个自定义页面和一个错误码，来替换一个不同的错误码响应或是重定向浏览器到一个不同的URL。下面例子中，用`error_page`指令指定404.html页面返回及404错误码

```nginx
error_page 404 /404.html;
```
> 注意该指令并不意味着错误会立即返回当错误发生时。错误码可以来自代理服务器或在处理过程中发生的错误。

> 下面的例子说明当nginx找不到页面时，它用301代替404状态码并重定向到http:/example.com/new/path.html，该配置非常有用当客户端仍尝试访问一个老的页面时。301状态码告知浏览器该页面被永久的移除了，且需要用新的地址来代替旧的地址返回。

```nginx
location /old/path.html {
    error_page 404 =301 http:/example.com/new/path.html;
}
```

> 下面的配置是一个传递请求到后端服务的例子当一个文件没有找到的时候。因为这里没有指定相应的状态码在=号后面，而是从代理服务器那里返回了一个状态码。

```nginx
server {
    ...
    location /images/ {
        # Set the root directory to search for the file
        root /data/www;

        # Disable logging of errors related to file existence
        open_file_cache_errors off;

        # Make an internal redirect if the file is not found
        error_page 404 = /fetch$uri;
    }

    location /fetch/ {
        proxy_pass http://backend/;
    }
}
```

该`error_page`指令指示nginx做一个内部跳转当文件没有找到的时候。而`open_file_cache_errors`指令表示当文件找不到时不记录错误信息，因为这里已经正确处理了。

## 静态文件服务
### 根目录与首页文件
> `root`指令指定了查找文件的根目录，为了得到请求文件的路径，nginx会把请求的URI添加到`root`指令参数后面，该指令可以放在`http`,`server`,`location`等任意级别的上下文中。

```nginx
server {
    root /www/data;

    location / {
    }

    location /images/ {
        autoindex on;
    }

    location ~ \.(mp3|mp4) {
        root /www/media;
    }
}
```
> 如果请求是以/结束的，则nginx把它当作是请求一个目录并在该目录下找到一个索引文件。`index`指令定义了索引文件的名字，为了让nginx返回一个自动生成的目录列表，可以添加`autoindex on`指令。

### 尝试多种选项
> `try_files`指令用于检查指定的文件或目录是否存在并做内部重定向，或是如果没有的话就返回一个指定的状态码。

```nginx
server {
    root /www/data;

    location /images/ {
        try_files $uri /images/default.gif;
    }
}
```
> 如果请求的源路径URL文件不存在，nginx会做一个内部重定向到最后一个参数指定的文件。

> 最后一个参数可以是一个状态码或是一个location名字，下面的例子中如果没有相应的文件和目录则返回404错误。

```nginx
location / {
    try_files $uri $uri/ $uri.html =404;
}
```

### nignx性能优化
> 加载速度是任何服务的一个关键因素。一些不重要的优化配置可能会提高你的工作效率，也可能会达到更好的性能。

**启用文件发送(sendfile)**

> 打开`sendfile`指令将省去copy数据到缓存buffer，而直接从一个文件copy数据到另一个文件，为防止一个连接占居所有的worker进程，可以设置限制一个sendfile的最大数据传输量（`sendfile_max_chunk`）

```nginx
location /mp3 {
    sendfile           on;
    sendfile_max_chunk 1m;
    ...
}
```

**启用tcp_nopush**
> 启用`tcp_nopush`选项与`sendfile on`结合，可以使nginx在通过sendfile获取数据后发送http响应头在一个包中。

```nginx
location /mp3 {
    sendfile   on;
    tcp_nopush on;
    ...
}
```

**启用tcp_nodelay**
> `tcp_nodelay`选项允许重写Nagle算法，该算法原本是解决在慢速网络中传输小的数据包的问题，该算法会把小包的数据集成为大包数据并有200ms的发送延迟。现在当请求大的静态文件时，数据可以立即发送而忽略包的大波。默认`tcp_nodelay`被设置为on，意味着关闭Nagle算法，常常仅用于保持活动连接。

```nginx
location /mp3  {
    tcp_nodelay       on;
    keepalive_timeout 65;
    ...
}
```

**优化Backlog队列**
> nginx处理接收的连接请求速度的一个重要因素就是backlog队列。一般是当一个连接建立时，它会放在监听队列里。在正常的压力下，这个队列有可能有很少的连接，也可能为空，但是在高负载情况下，队列可能会成倍的增长，导致不好的性能，连接可能会被扔掉。

> 可以调节操作系统的*net.core.somaxconn*的值，默认为128,来增加处理的峰值。如
`sudo sysctl -w net.core.somaxconn=4096`,然后设置nginx的backlog的值，如：

```nginx
server {
    listen 80 backlog 4096;
    # The rest of server configuration
}
```

## 反向代理

反射代理通常用于负载均衡，显示来自不同网站的内容或是传输请求到不同的服务器去处理

### 传递请求到代理服务器
> 这里有不同的转发代理指令，如下所示：

* proxy_pass    传递一个请求给一个http代理服务器
* fastcgi_pass    传递一个请求到fastcgi服务器
* uwsgi_pass    传递一个请求到uwsgi服务器
* scgi_pass    传递一个请求到scgi服务器
* memcached_pass    传递一个请求到memcached服务器

### 传输请求头
> nginx默认只在代理请求中重定义两个头信息Host=$proxy_host和Connection=close，可以使用`proxy_set_header`指令来修改或设置其他的头信息，该指令可以放在location,server或http上下文或块中，为了阻止头字段传给代理服务器，可以将其设置为空，如下所示：

```nginx
location /some/path/ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Accept-Encoding "";
    proxy_pass http://localhost:8000;
}
```

### 配置缓存buffers
> 默认情况下，nginx缓存来自代理服务器的响应，响应存储在内部buffer中直到所有的响应都收到才发送给客户端。开启buffer缓存，nginx允许代理服务器快速处理响应，同时nginx尽可能长时间的存储响应数据让客户端下载他们。

> 负责开启和关闭缓存命令是`proxy_buffering`，默认情况下是on，打开缓存的。而`proxy_buffers`指令则控制缓存的数量及大小，来自代理服务器的第一部分响应数据是存储在一个独立的缓存上的，其大小是通过`proxy_buffer_size`指令设置的，这部分通常包含一个比较小的响应报头，并且可以更小比存放响应其余部分的缓存。

```nginx
location /some/path/ {
    proxy_buffers 16 4k;
    proxy_buffer_size 2k;
    #proxy_buffering off;
    proxy_pass http://localhost:8000;
}
```

> 如果缓存关闭，则响应会被从代理服务器直接同步发送给客户端。这种情况可能需要与客户端快速交互且尽可能快的从收到响应就开始。如果关闭缓存，则nginx只会使用通过`proxy_buffer_size`设置的缓存来存储当前部分的响应数据。

### 选择哪些IP地址进行代理服务

> 如果你的代理服务器有几个网络接口，有时你可能需要选择一些特定的IP来与代理服务器进行交互，可能会需要配置接收来自特定IP网络或特定范围的IP时非常有用。通过`proxy_bind`指令来指定IP地址。

```nginx
location /app1/ {
    proxy_bind 127.0.0.1;
    proxy_pass http://example.com/app1/;
}

location /app2/ {
    proxy_bind 127.0.0.2;
    proxy_pass http://example.com/app2/;
}
```


## 压缩与解压缩
> 压缩响应数据在传输过程中，最大化的减少带宽的使用。然而，由于压缩发生在运行时它也可以增加相当大的处理开销而对性能产生负面影响。nginx不会对压缩过的数据再次进行压缩。

### 开启压缩
> 开启压缩则用`gzip on`指令即可。

```nginx
gzip on;
```

> 默认情况下，nginx仅会对MIME type text/html的响应进行压缩，如果要对其他形式的内容进行压缩，则用`gzip_types`指令。

```nginx
gzip_types text/plain application/xml;
```

> 指定压缩的响应的最小长度，可以使用`gzip_min_length`指令，默认是20bytes.

```nginx
gzip_min_length 1000;
```

> 默认情况下，nginx不会压缩代理请求的响应(来自代理服务器的请求)。事实上来自代理服务器的请求通过请求头**Via**的形式来决定，可以用`gzip_proxied`指令来指定哪些代理请求应该被压缩。比如压缩哪些没有被代理服务器缓存的响应是合理的。在这种情况下，`gzip_proxied`指令使nginx**Cache-Control`响应头，如果是**no-cache,no-store,or private**，就进行压缩，另外你必须包含**expired**参数去检索**Expires**头的值。

```nginx
gzip_proxied no-cache no-store private expired auth;
```

> 大部分的压缩指令可以放在**http**上下文或**server** or **location**配置块中，如下所示：

```nginx
server {
    gzip on;
    gzip_types      text/plain application/xml;
    gzip_proxied    no-cache no-store private expired auth;
    gzip_min_length 1000;
    ...
}
```



### 开启解压缩
> 一些客户端不支持压缩后的内容，这种情况下，更希望存储响应数据，或压缩数据在fly上并存储到缓存，为了更好的服务于支持与不支持压缩的客户端，nginx可以解压数据在传送时并发送给不支持压缩的客户端。

```nginx
server {
    gzip on;
    gzip_min_length 1000;
    gunzip on;
    ...
}
```

### 发送压缩文件
> 发送压缩后的文件给客户端.

```nginx
location / {
    gzip_static on;
}
```

> 在这种情况下，服务一个**/path/to/file**的请求时，nginx会尝试查找并发送**/path/to/file.gz**文件，如果该文件不存在，或客户端不支付gzip，则nginx会发送没有压缩的文件。需要注意的是，`gzip_static`指令不会开启实时压缩，它仅仅用于通过压缩工具压缩后的文件，要实时的压缩内容或静态文件，请使用`gzip`指令。

## Web内容缓存
> 缓存来自代理服务器的静态或动态内容。如果开启了缓存，则nginx会保存响应在磁盘缓存中，并用他们发送给客户端而不再每次都去请求代理服务器。

### 开启响应缓存
> 开启缓存，在http上下文包含`proxy_cache_path`指令即可，第一个必选参数是缓存存放的本地文件路径，必选的**key_zone**参数指定用于缓存元数据的缓存区域的名字和大小。而`proxy_cache`指令则指定了你要缓存响应时用的哪个缓存区块，即**keys_zone**的值。

```nginx
http {
    ...
    proxy_cache_path /data/nginx/cache keys_zone=one:10m;

    server {
    proxy_cache one;
    location / {
            proxy_pass http://localhost:8000;
        }
    }
}
```

> 注意这里__keys_zone__参数指令的大小并不是缓存响应数据的总大小，如果要缓存响应数据的总数，可以用**proxy_cache_path**指令的**max_size**参数指定。

### 涉及到cache的nginx处理
> 这里有两种情况会涉及到缓存处理：

* *cache manager*被激活去检查缓存的状态。如果缓存大小超过了**max_size*设置的大小，则缓存管理器会删除最近最少使用的缓存数据。如前面提到的，高速缓存的数据的量可以在高速缓存管理器激活之间的时间暂时超过极限。
* *cache loader*缓存管理器在nginx启动之后只会运行一次，它会加载之前缓存的元数据到共享的缓存区块。加载整个缓存一次性会消耗充分多的资源从而会降低nginx的性能，为了避免这种情况，可以通过配置`proxy_cache_path`指令的以下参数来迭代缓存的加载。
    * loader_threshold - 迭代的时间，默认是200毫秒
    * loader_files - 一次迭代加载的最大量，默认为100
    * loader_sleeps - 迭代周期，即时间间隔，默认为50ms
    
> 下面的例子就是迭代持续300ms或直到加载了200条数据为止：

```nginx
proxy_cache_path /data/nginx/cache keys_zone=one:10m loader_threshold=300 loader_files=200;
```

### 指定哪些请求被缓存
> 默认情况下，nginx会缓存所有http头是GET和HEAD方法请求的响应在第一次从代理服务器收到响应时。而nginx会用请求的request string做为请求缓存的key。指定cache key可以用`proxy_cache_key`指令：   
```nginx
proxy_cache_key "$host$request_uri$cookie_user";
```   
定义一个key重复出现多少次时就应该缓存这条请求的响应数据用`proxy_cache_uses`指令：   
```nginx
proxy_cache_min_uses 5;
```   
指定哪些方法的请求的响应参与缓存使用`proxy_cache_methods`指令：   
```nginx
proxy_cache_methods GET HEAD POST;
````

### 清除缓存
> nginx会从缓存清除过期的缓存文件，这是必须的，主了防止同时服务老的与新版的网页。当收到一个指定的*purge*请求，其他包含要么是自定义的HTTP头要么是PURGE HTTP method的，会被清除缓存。   

**配置缓存清除**

1. 在http这一层新建一变量，例如**$purge_method**, 它将依赖于**$request_method**变量：
    ```nginx
    http {
        ...
        map $request_method $purge_method {
            PURGE 1;
            default 0;
        }
    }
    ```

2. 在location里进行缓存配置，用`proxy_cache_purge`指令指定一个缓存清除的条件，如下所示，其中**$purge_method**已经在前定义了：

    ```nginx
    server {
        listen      80;
        server_name www.example.com;

        location / {
            proxy_pass  https://localhost:8002;
            proxy_cache mycache;

            proxy_cache_purge $purge_method;
        }
    }
    ```


**发送清除命令**

> 当使用PURGE方法去请求时，就会删除指定请求的缓存内容。如下所示：
    ```bash
    $ curl -X PURGE -D – "https://www.example.com/*"
    HTTP/1.1 204 No Content
    Server: nginx/1.5.7
    Date: Sat, 01 Dec 2015 16:33:04 GMT
    Connection: keep-alive
    ```

> 在上面的例子中，有相同url部分的资源会被删除，但这些缓存不会被从缓存中彻底的删除当不活动时，要么被缓存清除器处理时，要么被客户端试图访问时。

**彻底从缓存删除文件**

> 为了彻底删除缓存内容，你需要激活一个缓存清除处理进程永久的迭代的去删除匹配key的缓存内容。在http层为`proxy_cache_path`指令添加`purge`参数。
    ```nginx
    proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=mycache:10m purger=on;
    ```

**缓存清除配置样例**

```nginx
http {
    ...
    proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=mycache:10m purger=on;

    map $request_method $purge_method {
        PURGE 1;
        default 0;
    }

    server {
        listen      80;
        server_name www.example.com;

        location / {
            proxy_pass        https://localhost:8002;
            proxy_cache       mycache;
            proxy_cache_purge $purge_method;
        }
    }
}
```

### 限制或绕过缓存
> 一般情况下，只有超缓存最大数时，缓存才会被删除，但可以通过`proxy_cache_valid`指令去限制指定响应状态码的响应被缓存多少时间是合法的：
    ```nginx
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404      1m;
    ```

> 上面的例子中指定200,302的响应是合法的，会被缓存10分钟，如果要指定所有状态码的响应的合法缓存时间，则指定第一个参数为any即可：
    ```nginx
    proxy_cache_valid any 5m;
    ```

> 可以定义哪些条件下不发送缓存响应给客户端，使用`proxy_cache_bypass`指令，其每一个条件是由一组变量组成，如果至少有一个参数不为空或不为0,则nginx不从缓存里查找响应数据，直接发送请求给备用服务处理。
    ```nginx
    proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
    ```

> 使用`proxy_no_cache`指令参数指定哪些条件下一缓存响应数据：
    ```nginx
    proxy_no_cache $http_pragma $http_authorization;
    ```

### 组合配置例子

```nginx
http {
    ...
    proxy_cache_path /data/nginx/cache keys_zone=one:10m loader_threshold=300 
    loader_files=200 max_size=200m;

    server {
        listen 8080;
        proxy_cache one;

        location / {
            proxy_pass http://backend1;
        }

        location /some/path {
            proxy_pass http://backend2;
            proxy_cache_valid any 1m;
            proxy_cache_min_uses 3;
            proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
        }
    }
}
```

> 在这个例子中两个location使用了不同的缓存方式。   
因为来自**backend1**的响应很少变，且没有缓存相应的指定，响应一般在第一次请求时被缓存，且始终有效。  
相对来说，来自**backend2**的响应经常变化，所以被设置为1分钟有效，且直接相同的请求出现3次才会被缓存，如果请求与`proxy_cache_bypass`指令匹配，则立即发送请求到**backend2**服务器而不再进行缓存查找。

***

## 管理SSL事务

### SSL终端
> 通过HTTPS发送web内容

**设置一个HTTPS 服务器**

> 通过`listen`指令的ssl参数配置一个HTTPS server，然后再配置服务器证书及私钥文件的位置，如：


```nginx
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ...
}
```

> 服务器证书是公共的，会发给连接服务口拙所有客户端，而那个安全私钥应该放在一个限制访问的文件内，但nginx的master进程可以读取，有时私钥会和证书放在同一个文件内：


```nginx
ssl_certificate www.example.com.cert;
ssl_certificate_key www.example.com.cert;
```

> `ssl_protocols`指令和`ssl_ciphers`指令用来限制只包括强大的版本和SSL/ TLS的密码的连接。因为1.0.5的nginx默认使用**ssl_protocols SSLv3 TLSv1 and ssl_ciphers HIGH:!aNULL:!MD5**,而1.1.13和1.0.12版本默认升级到**ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2**


**HTTPS server 优化**

> SSL操作会消耗额外的CPU资源，而最点CPU的操作就是SSL握手。下面有两种方式可以减少每个客户端的这些操作：

* 启用保持活动的连接通过一个连接发送多个请求
* 重用SSL会话参数，以避免并行和后续连接SSL握手

> 会话存储在SSL会话缓存中，被worker进程所共享，通过`ssl_session_cache`指令进行设置。一个cache包含大概4000的会话。默认的cache过期时间是5分钟，可以用`ssl_session_timeout`指令来设置过期时间，如下所示：

```nginx
worker_processes auto;

http {
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;

    server {
        listen              443 ssl;
        server_name         www.example.com;
        keepalive_timeout   70;

        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        ...
    }
}
```

**SSL证书链**

> 可以用证书链顺序的整合证书到一个文件里，然后用`ssl_certificate`指令进行设置,如果服务证书绑定顺序出错，则nginx会启动失败并报错:

```bash
$ cat www.example.com.crt bundle.crt > www.example.com.chained.crt
```

```nginx
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.chained.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```

**单独的一个HTTP/HTTPS服务器**

> 有可能会设置一个服务器来同时处理HTTP和HTTPS请求，在一个server里通过设置一个带ssl参数的`listen`指令，一个没有带ssl参数：

```nginx
server {
    listen              80;
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```

**基于HTTPS的多个服务器**

> 两个或多个HTTPS服务器监听相同的IP地址：


```nginx
server {
    listen          443 ssl;
    server_name     www.example.com;
    ssl_certificate www.example.com.crt;
    ...
}

server {
    listen          443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    ...
}
```

> 在这种配置下浏览器会接收默认服务器的证书，而忽略请求的服务名server name，这是由SSL协议行为造成的，解决这种问题最好的方式是为每一个HTTPS服务器分配独立的IP地址：


```nginx
server {
    listen          192.168.1.1:443 ssl;
    server_name     www.example.com;
    ssl_certificate www.example.com.crt;
    ...
}

server {
    listen          192.168.1.2:443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    ...
}
```

> 多个服务共享一个证书时，可以把证书的配置放在*http*这一层，以便所有的服务器继承一个单独的内存copy：


```nginx
ssl_certificate     common.crt;
ssl_certificate_key common.key;

server {
    listen          443 ssl;
    server_name     www.example.com;
    ...
}

server {
    listen          443 ssl;
    server_name     www.example.org;
    ...
}
```

### 针对TCP上游的SSL终端
> 通过HTTPS发送TCP网络

**什么是SSL终端**

> SSL终端是与客户端的连接的服务一方的一个SSL端点，它执行后端服务器必须要做的对请求的解密及对响应的加密工作。该操作被称为终结,因为nginx会关闭客户端连接，并新建转发的数据。

**前提条件**

* nginx 版本R6及更高版本
* 有几个TCP服务器的负责均衡上游组
* SSL 证书和一个私钥

**获取SSL证书**

> 首先你得获取服务器证书和一个私钥，证书可以从一个CA机构获取或是用SSL库如OpenSSL生成。

**配置nginx**

1. 开启SSL
    
    ```nginx
    stream {
         
        server {
            listen     12345 ssl;
            proxy_pass backend;
            …
        }
    }
    ```

2. 添加SSL证书

    > 添加证书，指定证书的位置用`ssl_certificate`指令，用`ssl_certificate_key`指令指定私钥的位置。

    ```nginx
    server {
        …
        ssl_certificate        /etc/ssl/certs/server.crt;
        ssl_certificate_key    /etc/ssl/certs/server.key;
    }
    ```

    > 另外用`ssl_protocols`和`ssl_ciphers`指令来限制和SSL/TLS的版本及密码：

    ```nginx
    server {
        …
        ssl_protocols  SSLv3 TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers    HIGH:!aNULL:!MD5;
    }
    ```

**加快安全的TCP连接**

> 实现SSL/TLS会很影响服务器性能，因为SSL握手操作很占CPU资源，一般SSL握手超时时间为60s,但可以用`ssl_handshake_timeout`指令去修改，我们不建议设置太高或太低的值，因为很容易导握手失败或长时间等待等。

```nginx
server {
    …
    ssl_handshake_timeout 10s;
}
```

* 优化SSL会话缓存

    > 创建会话参数的缓存可以使SSL/TLS连接减少握手的次数，从而提高性能，缓存用`ssl_session_cache`指令来设置：

    ```nginx
    ssl_session_cache;
    ```

    > 默认情况下nginx会用内置的会话缓存，就是说缓存是建立在你的SSL库里的。这通常是不可选的，且只能被一个worker进程使用，容易造成内存碎片，可以通过`ssl_session_cache`指令来设置共享缓存在所有的worker进行间，这样可以加快连接：

    ```nginx
    ssl_session_cache shared:SSL:1m;
    ```

    > 作为参考，1M的缓存大概可以容纳4000的会话。

    > 默认情况下，nginx保留缓存的会话参数5分钟，增加`ssl_session_timeout`指令的值到几人小时可以拉高性能，因为重复使用缓存会话可以减少握手次数，同时你增加了超时时间，也得同步增大缓存到适合的容量：

    ```nginx
    server {
        …
        ssl_session_cache   shared:SSL:20m;
        ssl_session_timeout 4h;
    }
    ```

* 会话票据

    > 会话票据是对会话缓存的一种替代，会话信息被缓存在客户端，省去了服务器端缓存的需要，录客户端与服务器再次交互时，它使用会话票据即可，不再需要谈判：

    ```nginx
    server {
        …
        ssl_session_tickets on;
    }
    ```

    > 当针对一组上游服务器使用会话票据时，每一个上游服务都必须用相应的会话key进行初始化，这样改变会话key会很方便：

    ```nginx
    server {
        …
        ssl_session_tickets on;
        ssl_session_ticket_key /etc/ssl/session_ticket_keys/current.key;
        ssl_session_ticket_key /etc/ssl/session_ticket_keys/previous.key;
    }
    ```

**完整例子**

```nginx
stream {
    upstream stream_backend {
        server backend1.example.com:12345;
        server backend2.example.com:12345;
        server backend3.example.com:12345;
   }

    server {
        listen                12345 ssl;
        proxy_pass            stream_backend;
         
        ssl_certificate       /etc/ssl/certs/server.crt;
        ssl_certificate_key   /etc/ssl/certs/server.key;
        ssl_protocols         SSLv3 TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers           HIGH:!aNULL:!MD5;
        ssl_session_cache     shared:SSL:20m;
        ssl_session_timeout   4h;
        ssl_handshake_timeout 30s;
        …
    }
}
```


### 在nginx和HTTP上游服务之间的SSL
> 保护nginx和上游服务器之间的HTTP流量

**前提条件**

* nginx开源代码
* 一个代理服务器或由一组服务器组成的上游服务
* SSL证书和一个私钥

**获取SSL服务器证书**

> 你可以购买一个服务器证书从CA，或你可用用一个OpenSSL库创建自己内部的CA并生成自己的证书，服务器证书和私钥应该放在每一个上游服务器上。

**获取SSL客户端证书**

> 你要获取一个客户端证书，且需要配置上游服务器，对所有进来的SSL连接都要客户端证书，然后当nginx连接上游服务器时，它会提供客户端的证书同时上游服务器会接受它。

**配置nginx**

> 首先改变url到一组支持SSL连接的上游服务器，在配置文件中，用`proxy_pass`指令为代理服务器指定使用https协议：

```nginx
location /upstream {
    proxy_pass https://backend.example.com;
}
```

> 添加客户端证书和key用于验证nginx在每一个上游服务器上：

```nginx
location /upstream {
    proxy_pass                https://backend.example.com;
    proxy_ssl_certificate     /etc/nginx/client.pem;
    proxy_ssl_certificate_key /etc/nginx/client.key
}
```

> 如果你使用自签名证书针对一个上游服务器或你自己的CA，可以包含`proxy_ssl_trusted_certificate`指令，但文件必须是PEM格式，同时也可以包含`proxy_ssl_verify`和`proxy_ssl_verfiy_depth`指令来检验安全证书的有效性：

```nginx
location /upstream {
    ...
    proxy_ssl_trusted_certificate /etc/nginx/trusted_ca_cert.crt;
    proxy_ssl_verify       on;
    proxy_ssl_verify_depth 2;
    ...
}
```

> 每次SSL握手都要占用CPU资源，影响性能，可以使用`proxy_ssl_session_reuse`指令来简化的握手，同时你也可以指定哪些SSL协议和密码被使用：

```nginx
location /upstream {
    ...
    proxy_ssl_session_reuse on;
    proxy_ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    proxy_ssl_ciphers   HIGH:!aNULL:!MD5;
    ...
}
```

**配置上游服务器**

> 每一个上游服务器应该配置接受HTTPS连接，为每一个上游服务指定一个服务器证书及私钥的路径：

```nginx
server {
    listen              443 ssl;
    server_name         backend1.example.com;

    ssl_certificate     /etc/ssl/certs/server.crt;
    ssl_certificate_key /etc/ssl/certs/server.key;
    ...
    location /yourapp {
        proxy_pass http://url_to_app.com;
        ...
    }
}
``` 

> 指定客户端证书路径：

```nginx
server {
    ...
    ssl_client_certificate /etc/ssl/certs/ca.crt;
    ssl_verify_client      off;
    ...
}
```

**完整例子**

```nginx
http {
    ...
    upstream backend.example.com {
        server backend1.example.com:443;
        server backend2.example.com:443;
   }

    server {
        listen      80;
        server_name www.example.com;
        ...

        location /upstream {
            proxy_pass                    https://backend.example.com;
            proxy_ssl_certificate         /etc/nginx/client.pem;
            proxy_ssl_certificate_key     /etc/nginx/client.key
            proxy_ssl_protocols           SSLv3 TLSv1 TLSv1.1 TLSv1.2;
            proxy_ssl_ciphers             HIGH:!aNULL:!MD5;
            proxy_ssl_trusted_certificate /etc/nginx/trusted_ca_cert.crt;

            proxy_ssl_verify        on;
            proxy_ssl_verify_depth  2;
            proxy_ssl_session_reuse on;
        }
    }

    server {
        listen      443 ssl;
        server_name backend1.example.com;

        ssl_certificate        /etc/ssl/certs/server.crt;
        ssl_certificate_key    /etc/ssl/certs/server.key;
        ssl_client_certificate /etc/ssl/certs/ca.crt;
        ssl_verify_client      off;

        location /yourapp {
            proxy_pass http://url_to_app.com;
        ...
        }

    server {
        listen      443 ssl;
        server_name backend2.example.com;

        ssl_certificate        /etc/ssl/certs/server.crt;
        ssl_certificate_key    /etc/ssl/certs/server.key;
        ssl_client_certificate /etc/ssl/certs/ca.crt;
        ssl_verify_client      off;

        location /yourapp {
            proxy_pass http://url_to_app.com;
        ...
        }
    }
}
```

> 该例子中指定到上游服务器的请求使用HTTPS协议。当一个安全连接第一次被发送到上游服务器时，会执行全过程的握手程序。而` proxy_ssl_certificate`指令定义了PEM格式文件的路径，` proxy_ssl_certificate_key`指令定义了私钥的路径，而` proxy_ssl_protocols and proxy_ssl_ciphers `指令控制哪些协议和密码被使用。   
当连接请求再次过来时，会话参数将会重复使用通过`proxy_ssl_session_reuse`指令，则安全连接会更快的建立 。



### 在nginx和TCP上游之间的SSL
> 保护nginx和上游服务器之间的TCP流量

**前提条件**

* nginx R6版本或更高或最新的源码
* 一个TCP代理服务器或由一组TCP服务器组成的上游服务组
* SSL证书和私钥

**获取SSL服务器证书**

> 获取服务器证书及私钥并放在后端服务器上，证书可以从一个CA机构获取或是用SSL库如OpenSSL生成。如果你只需要加密NGINX和上游服务器之间的连接时可以使用自签名的服务器证书，然而这些连接很容易受到中间人的攻击，冒名顶替者可以冒充上游服务器而NGINX不知道它是跟一个假的服务器在通信。如果你获得了来自信任的CA的服务器证书，就很难被攻击。

**获取客户端证书**

> 你也许要保证上游服务器的安全而只接收来自nginx的请求并不是攻击者。nginx可以用客户端证书来向上游服务器证实自己的身份，这些客户端证书必须是来自信任的CA的签名。同时你需要配置上游服务器对到来的请求都需要客户端证书，当nginx连接上上游服务时，它会提供自己的客户端证书，这时上游服务器才会接受它。

**配置nginx**

> 在nginx配置文件中，在stream层级的server块包含`proxy_ssl`指令：

```nginx
stream {
    server {
        ...
        proxy_pass backend;
        proxy_ssl  on;
    }
}
```

> 然后指定在后端服务器上证书及私钥的路径：

```nginx
server {
        ...
        proxy_ssl_certificate     /etc/ssl/certs/backend.crt;
        proxy_ssl_certificate_key /etc/ssl/certs/backend.key;
}
```

> 然后你可以指定使用哪些协议和密码：

```nginx
server {
        ...
        proxy_ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
        proxy_ssl_ciphers   HIGH:!aNULL:!MD5;
}
```

> 如果你使用由CA发布的证书，也可以包含由`proxy_ssl_trusted_certificate`指令命名包含信任证书的文件，来验证后端服务器的安全证书，该文件必须是PEM格式。可以包含`proxy_ssl_verify and proxy_ssl_verfiy_depth`指令来验证安全证书的有效性：

```nginx
server {
    ...
    proxy_ssl_trusted_certificate /etc/ssl/certs/trusted_ca_cert.crt;
    proxy_ssl_verify       on;
    proxy_ssl_verify_depth 2;
}
```

> 每次SSL握手都要占用CPU资源，影响性能，可以使用`proxy_ssl_session_reuse`指令来简化的握手:

```nginx
proxy_ssl_session_reuse on;
```

**完整例子**

```nginx
stream {

    upstream backend {
        server backend1.example.com:12345;
        server backend2.example.com:12345;
        server backend3.example.com:12345;
   }

    server {
        listen     12345;
        proxy_pass backend;
        proxy_ssl  on;

        proxy_ssl_certificate         /etc/ssl/certs/backend.crt;
        proxy_ssl_certificate_key     /etc/ssl/certs/backend.key;
        proxy_ssl_protocols           SSLv3 TLSv1 TLSv1.1 TLSv1.2;
        proxy_ssl_ciphers             HIGH:!aNULL:!MD5;
        proxy_ssl_trusted_certificate /etc/ssl/certs/trusted_ca_cert.crt;

        proxy_ssl_verify        on;
        proxy_ssl_verify_depth  2;
        proxy_ssl_session_reuse on;
    }
}
```

***

## 负载均衡

### HTTP负载均衡器
> 在一组服务器之间,基于一种选择算法被动或主动的去检索上游服务的健康性及运行时的负载均衡的配置修改等，并选择服务器进行分发HTTP请求，

**概述**

> 跨多个应用程序实例的负载均衡是一个常用的技术，多用于优化资源利用、最大化吞吐量、减少等待时间以及确保容错配置等,NGINX可以用作在不同的部署方案下非常有效的HTTP负载平衡器。

**一组服务器的代理业务**

> 首先你需要用`upstream`指令定义组，同时用`server`指令定义组中的多个服务器：

```nginx
http {
    upstream backend {
        server backend1.example.com weight=5;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
}
```

> 然后发送所有的请求到代理服务器组：

```nginx
server {
    location / {
        proxy_pass http://backend;
    }
}
```

**选择一种负载均衡方法**

> NGINX支持4种负载均衡方法：

1. 轮循方法：所有的请求都会根据权重均匀的分布在各个服务器上，该方法是默认的方法，不需要用指令开启开。

    ```nginx
    upstream backend {
       server backend1.example.com;
       server backend2.example.com;
    }
    ```

2. 最少连接方法：一个请求会被分发到根据权重的最少连接的服务器上：

    ```nginx
    upstream backend {
        least_conn;

        server backend1.example.com;
        server backend2.example.com;
    }
    ```

3. ip哈希方法：一个请求被分发到哪个服务器决定于客户端的IP地址，根据IP地址来计算其哈希值，来自相同IP地址的请求会被分发到相同的服务器上。

    ```nginx
    upstream backend {
        ip_hash;

        server backend1.example.com;
        server backend2.example.com;
    }
    ```
        
    > 如果一个服务器需要临时删除，则给其后添加`down`参数，以便保留当前客户端IP的哈希值，则请求会被自动发送到下一个服务器进行处理：

    ```nginx
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com down;
    }
    ```

4. 通用散列法：其请求分发决定于用户自定义的hash key，可以是文本，变量，或是它们的组合，比如IP地址和端口号，或是URI：

    ```nginx
    upstream backend {
        hash $request_uri consistent;

        server backend1.example.com;
        server backend2.example.com;
    }
    ```

    > 其中*consistent*可选参数会开启 ketama 哈希负载均衡，如果上游服务器组添加或删除服务器，会有一小部分key将被重新映射，减少缓存未命中率。

5. 最小次数访求：对于每个请求，NGINX选择具有最低平均等待时间和数量最少的活动连接的服务器处理，而最低的平均等待时间是基于以下参数来计算的：

    * header - 从服务器收到第一个字节的时间
    * last_byte - 从服务器收到所有响应的时间

    ```nginx
    upstream backend {
        least_time header;

        server backend1.example.com;
        server backend2.example.com;
    }
    ```

**服务器权重**

> 默认情况下，nginx使用轮循法根据权重分发请求到相应的服务器，可以通过server指令的`weight`参数指定权重，默认权重为1：

```nginx
upstream backend {
        server backend1.example.com weight=5;
            server backend2.example.com;
                server 192.0.0.1 backup;
}
```

> 其中一个服务器被标记为备用服务器，该服务器不会接收请求，除非其他两个服务器不可用。

**服务器慢启动**

> 服务器慢启动功能可以防止最近从被连接淹没中恢复的服务器，这可能会超时并导致服务器被标记为再次失败。   
慢启动可以使一个上游服务器渐渐的恢复它的权重从0开始，直到变的可用，可以用`slow_start`参数完成。

```nginx
upstream backend {
    server backend1.example.com slow_start=30s;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

> 时间值设置了服务器装恢复权重的时间。如果该组里只有一个单一的服务器，则`max_fails`,`fail_timeout`和`slow_start`参数将不起作用，这个服务器将永远都不会被认为是不可用的。

**启用会话持久性**

> 会话持久性意味着nginx标识用户会话并路由来自该会话的请求到相应的上游服务器。nginx支持三种会话持久性方法，这些方法通过`sticky`指令来设置。

* `sticky cookie`方法。该方法,nginx会添加会话cookie到来自上游服务器组的第一个响应中并标识该服务器已发送过响应，当客户端发送下一个请求时，它将会包含该cookie值，且nginx会路由该请求到相同的上游服务器：

    ```nginx
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;

        sticky cookie srv_id expires=1h domain=.example.com path=/;
    }
    ```

* `sticky route`方法。该方法会使用nginx为客户端分配一个路由当它收到第一个请求时，而所有的子请求将会与`server`指令的`route`参数做比较来确定哪些请求将会被代理。路由信息可以来自cookie或URI。

    ```nginx
    upstream backend {
        server backend1.example.com route=a;
        server backend2.example.com route=b;

        sticky route $route_cookie $route_uri;
    }
    ```

* `cookie learn`方法。在该方法中nginx会首先会通过检查请求和响应来查找会话标识。然后nginx会发现哪些上游服务器与哪些会话标识通信。一般的这些标识会在cookie里传递。如果一个请求包含了已被学习过的会话标识，nginx会转发请求到相应的服务器。

    ```nginx
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;

        sticky learn 
        create=$upstream_cookie_examplecookie
        lookup=$cookie_examplecookie
        zone=client_sessions:1m
        timeout=1h;
    }
    ```

    在该例子中，*create*参数指定了一个变量，用来说明一个新的会话是怎么被创建的。我们的例子中，新会话是通过上游服务器的cookie中的*EXAMPLECOOKIE*创建的。   
    *lookup*参数说明怎么查找已存在的会话。例子中是在客户端发送的cookie里的*EXAMPLECOOKIE*查找存在的会话。   
    *zone*参数指定了一个共享的缓存区域，所有的会话都保存在这里。   
    该方法比较复杂，它不会需要保持客户端的任何cookie，所有的信息都保存在服务端的共享缓存里。

**限制连接数**

> 通过`max_conns`参数可以设置nginx期望的最大连接数。如果*max_conns*达到了最大值，则通过`queue`指令可以使请求被放到相应的队列中。该指令指定了队列中可以同时存在的最大请求数：

```nginx
upstream backend {
    server backend1.example.com  max_conns=3;
    server backend2.example.com;

    queue 100 timeout=70;
}
```

> 如果队列已经填满，或等待超时，则客户端将收到一个错误信息。如果在别的worker进程中有保持活动的连接则*max_conns*参数将会忽略。

**被动式健康监测**

> 当一个服务器不可用时，它将会停止发送请求到相应的服务器直到可用为止。下面参数配置在哪些情况下认为服务器不可用：

* `fail_timeout`参数指定了一段时间，在期间尝试连接失败指定数目时，就认为该服务器仍不可用。
* `max_fails`参数指定了尝试的次数。

> 默认情况下是10s尝试一次，所以如果nginx发送请求或接收响应失败一次，则认为服务器10s内不可用：

```nginx
upstream backend {                
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
    server backend3.example.com max_fails=2;
}
``` 

**主动式健康监测**

> 周期性地发送特殊的请求给每个服务器并检查满足一定的条件下可以监控服务器的可用性的回应。   
在发送请求给相应的上游组的location区块里要包含*health_check*指令，而服务器组应该用*zone*指令来配置：

```nginx
http {
    upstream backend {
        zone backend 64k;

        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
        server backend4.example.com;
    }

    server {
        location / {
            proxy_pass http://backend;
            health_check interval=10 fails=3 passes=2 uri=/some/path;
        }
    }
}
```

> 该例子开启了默认的健康监控。nginx会每5s发送"/"请求到每一个备用服务器。如果通信错误或超时，则认为该代理服务器是不健康的直到下一次检查。*zone*指令则定义了一个缓存区域，用来worker进程共享使用存在服务器组的配置信息。该例子是每10s检查一次，失败3次，通过2次则可以认为是健康的。也可以指定URI的请求进行健康检查。

> 最后，可以自定义健康条件，该条件用*match*块来指定。

```nginx
http {
    ...

    match server_ok {
        status 200-399;
        body !~ "maintenance mode";
    }

    server {
        ...

        location / {
            proxy_pass http://backend;
            health_check match=server_ok;
        }
    }
}
```

> `match`指令允许nginx检查状态，头字段及响应内容，该指令也可以包含一个状态条件、一个响应体条件和多个响应头条件，且响应必须满足所有指定的条件才被认为是健康的。

**与多个工作进程共享数据**

> 如果`upstream`指令没有包含*zone*指令，则每一个worker进程都会保持自己的服务器配置copy信息来维持自己相应的计数器，包括每一个服务器当前连接的数量和失败尝试发送请求到服务器的数量。   
如果包含*zone*指令，则服务器组的配置信息会放在一个所有worker进程共享的缓存里，这种情况是动态配置，因为worder进程组会访问组配置相同的复本，及使用相同的计数器。

**使用DNS配置HTTP负载均衡**

> nginx可以监控相应域名对应的IP地址的变化，同时自动的应用这些变化而不需要重新启动。这些可以用`resolver`指令来完成，该指令必须在*http*块中指定，在服务器组中指定`server`指令的*resolve*参数：

```nginx
http {
    resolver 10.0.0.1 valid=300s ipv6=off;
    resolver_timeout 10s;

    server {
        location / {
            proxy_pass http://backend;
        }
    }
           
    upstream backend {
        zone backend 32k;
        least_conn;
        ...
        server backend1.example.com resolve;
        server backend2.example.com resolve;
    }
}
```

> 默认情况下，nginx会基于TTL来解析DNS，但TTL的值可以通过*valid*参数进行重写，如下例中的5分钟。而*ipv6=off*参数则说明只解析IPV4地址。如果一个域名解析到多个IP地址，这些地址将被保存到上游配置中参与负载均衡，上例中这些服务器根据*least_conn*方法进行均衡。如果一个或多个IP地址被添加或删除，则服务器将重新平衡。





### TCP负载均衡器
> 在一组服务器之间,基于一种选择算法被动或主动的去检索上游服务的健康性及运行时的负载均衡的配置修改等，并选择服务器进行分发TCP连接，

**概述**

> 在版本5及之后的nginx可以代理及负载均衡TCP流量，TCP是各种流行应用及服务的一种协议，如LDAP，MySQL和RTMP。

> TCP负载均衡由三个nginx模块实现：

* ngx_stream_core_module - 字义了开启TCP流量处理的基本指令
* ngx_stream_proxy_module - 定义了代理TCP流量的指令
* ngx_stream_upstream_module - 定义了TCP负载均衡的指令

> 前提条件

* 版本5及之后的nginx
* 一个应用，数据库或服务通过TCP通信
* 后端服务器，每一个应用，数据库或服务的运行实例

**配置TCP负载均衡**

> 负载均衡可以有效地分发网络流量到多个后端服务器：

1. 在http上下文中创建一个高层级的*stream*上下文
2. 定义由多个服务器组成的上游组来负载平衡
3. 定义一个服务器监听TCP连接并代理他们到上游服务器组
4. 为每一个服务器配置负载均衡方法及参数包含最大连接数、服务权重等

*创建一个上游服务器组*

```nginx
stream {
    upstream stream_backend {
        server backend1.example.com:12345 weight=5;
        server backend2.example.com:12345;
        server backend3.example.com:12346;
    }
}
```

*代理TCP流量*

```nginx
stream {
    ...
    server {
        listen     12345;
        proxy_pass stream_backend;
    }
}
```

*选择通过代理的IP地址*

> 如果你的代理服务器有几个网络接口，有时你需要选择一特定的IP地址与代理服务器或上游服务连接，这将很有用在指定IP地址或IP范围内接收连接，指定IP地址可以通过`proxy_bind`指令实现：

```nginx
stream {
    ...
    server {
        listen     127.0.0.1:12345;
        proxy_pass backend4.example.com:12345;
        proxy_bind 127.0.0.1:12345;
    }
}
```
    
*额外的配置*

> nginx有两内在缓冲区，用来旋转客户端数据或上游连接。一旦收到一个连接的数据，nginx就读取他并在别的连接上发送他，缓冲区可以由`proxy_buffer_size`指令来控制。如果数据比较少，则缓冲区可以减小点用来存放缓存资源，如果数据量很大，则缓冲区可以增加而减少socket读写操作数：

```nginx
stream {
    ...
    server {
        listen            127.0.0.1:12345;
        proxy_buffer_size 16k;
        proxy_pass        backend4.example.com:12345;
    }
}
```

**选择一种负载均衡方法**

> 配置不同的负载均衡算法，在*upstream*配置块中包含以下指令：

* least_conn - 为每个请求，nginx选择当前最少活动连接数的服务器
    
    ```nginx
    upstream stream_backend {
        least_conn;

        server backend1.example.com:12345 weight=5;
        server backend2.example.com:12345;
        server backend3.example.com:12346;
    }
    ```

* least_time - 对于每个连接，NGINX加选择服务器具有最低平均等待时间和数量最少的活动连接，在那里最低的平均等待时间是基于以下参数来计算被包括在least_time指令中：
    
    * connect - 连接上游服务器的时间
    * first_byte - 收到第一字段数据的时间
    * last_byte - 收到全部响应数据的时间

    ```nginx
    upstream stream_backend {
        least_time first_byte;

        server backend1.example.com:12345 weight=5;
        server backend2.example.com:12345;
        server backend3.example.com:12346;
    }
    ```

* hash - nginx基于用户自定义key的服务器，key可以是IP地址:

    ```nginx
    upstream stream_backend {
        hash $remote_addr consistent;

        server backend1.example.com:12345 weight=5;
        server backend2.example.com:12345;
        server backend3.example.com:12346;
    }
    ```

    > *consistent*可选参数应用ketama一致的散列方法，确保当服务器从上游组中添加或删除时对到其他服务器的key的重新映射次数。

**配置会话持久性**

> 配置持久性，则用*hash*负载均衡访求，因为该方法是基于客户端IP地址的，来自一个给定的客户端的TCP请求总是会被发送到相同的服务器除非该服务器不可用。

**限制连接数**

```nginx
upstream stream_backend {
    server backend1.example.com:12345 weight=5;
    server backend2.example.com:12345;
    server backend3.example.com:12346 max_conns=3;
}
```

**被动健康监控**

> 以下参数可以定义在哪些条件下认为服务器不可用：

* `fail_timeout` - 参数指定了一段时间，在期间尝试连接失败指定数目时，就认为该服务器仍不可用。
* `max_fails` - 参数指定了尝试的次数.

> 默认情况下是10s尝试一次，所以如果nginx发送请求或接收响应失败一次，则认为服务器10s内不可用

```nginx
upstream stream_backend {
    server backend1.example.com:12345 weight=5;
    server backend2.example.com:12345 max_fails=2 fail_timeout=30s;
    server backend3.example.com:12346 max_conns=3;
}
```

**主动健康监控**

> nginx发送特定的健康检查请求到每一个TCP上游服务器并检查响应是否满足特定条件，如果到一个服务器的连接没有建立，则健康检查失败，且该服务器被认为不可用。nginx不会代理客户端连接到不健康的服务器、如果健康检查是为一组服务器定义的，则只要有一个检查失败，则认为相应的服务器不可用。

*前提条件*

* 你必须配置一个上游服务器组在*stream*上下文中：

    ```nginx
    stream {
        upstream stream_backend {
             
            server backend1.example.com:12345;
            server backend2.example.com:12345;
            server backend3.example.com:12345;
       }
    }
    ```

* 你必须配置一个服务器传送TC决不能服务器组：

    ```nginx
    server {
        listen     12345;
        proxy_pass stream_backend;
    }
    ```

*基本配置*

1. 添加*zone*指令到上游服务器组中，指定区域的名字分配内存的大小，内存区块给所有的worker进程共享信息：

    ```nginx
    stream {
        upstream stream_backend {

            zone   stream_backend 64k; 
            server backend1.example.com:12345;
            server backend2.example.com:12345;
            server backend3.example.com:12345;
       }
    }
    ```

2. 添加`health_check`和`health_check_timeout`指令为代理连接到上游服务的服务器：

    ```nginx
    server {
        listen 12345;
        proxy_pass stream_backend;
        health_check;
        health_check_timeout 5s;
    }
    ```

*微调健康检查*

> 默认情况下，nginx每5秒尝试连接上游服务器组中的每一个服务器，如果连接失败，则认为检查不成功，同时标记相应的服务器不可用，并停止相应的客户端连接到服务器，要改变默认的行为，可以包含`health_check`指令的以下参数来实现：

* **interval** - 多长时间nginx发送健康检查请求，默认是5秒
* **passes** - 服务器必须响应的健康的连续的数量，才被认为是健康的，默认是1次
* **fails** - 连续失败多少次认为是不健康的，默认是1次

```nginx
server {
    listen       12345;
    proxy_pass   stream_backend;
    health_check interval=10 passes=2 fails=3;
}
```

*用匹配配置块微调健康检查*

> 用以下参数在**match**块中去定义成功健康检查条件：

* **send** - 发送给服务器的文本串
* **expect** - 需要匹配服务端返回的数据的正则表达式

> 这些参数可以以不同的组合使用，但一次最多包含一个send和expect参数。*send*和*expect*参数可以由如下的组合：

* 如果没有指定`send`或`expect`参数，以连接到服务器的能力进行测试
* 如果指定了预期参数，服务器预计将无条件地首先发送数据

    ```nginx
    match pop3 {
        expect ~* "\+OK";
    }
    ```

* 如果指定了`send`参数，则预计连接将会成功建立且指定的字段串被发送到服务器

    ```nginx
    match pop_quit {
            send QUIT;
    }
    ```

* 如果`send`和`expect`参数同时被指定，则`send`参数的字符串必须与`expect`参数的正则表达式匹配：

    ```nginx
    stream {
        upstream   stream_backend {
            zone   upstream_backend 64k;
            server backend1.example.com:12345;
        }
     
        match http {
            send      "GET / HTTP/1.0\r\nHost: localhost\r\n\r\n";
            expect ~* "200 OK";
        }
         
        server {
            listen       12345;
            health_check match=http;
            proxy_pass   stream_backend;
        }
    }
    ```

    > 例子表明，为了能够通过健康检查，则HTTP请求必须发送到服务器，并返回期望的结果中包含"200 OK"表明一个成功的HTTP响应

**即时配置**

> 上游服务器组可以使用简单的HTTP接口很容易的即时配置，用该接口，你可以浏览组中所有的服务器或一个特定的服务器，修改服务器参数，和添加或删除上游服务器。开启即时配置，你需要：

* 授权访问`upstream_conf`处理器，一个特定的处理器可以配置去监控和重配上游服务器，为了这样做，你需要在http块放置`upstream_conf`指令 在一个单独的location块内，限制该location的访问：

    ```nginx
    http {

        server {
            location /upstream_conf {
                upstream_conf;

                allow 127.0.0.1; # permit access from localhost
                deny all; # deny access from everywhere else
            }
        }
    }
    ```

* 在`stream`块，指定`zone`指令为上游服务器组，该指令可以创建一个共享的缓存区块为服务器组保存配置信息，以便所有的worker进程都可以使用相同的配置：

    ```nginx
    upstream stream_backend {
        zone backend 64k;
        ...
    }
    ```

*即时配置例子*

```nginx
stream {
        ...
        # Configuration of an upstream server group
        upstream appservers {
            zone appservers 64k;

            server appserv1.example.com:12345 weight=5;
            server appserv2.example.com:12345 fail_timeout=5s;

            server backup1.example.com:12345 backup;
            server backup2.example.com:12345 backup;
        }

        server {
            # Server that proxies connections to the upstream group
            proxy_pass appservers;
            health_check;
        }
}

http {
    …
    server {
        # Location for configuration requests
        location /upstream_conf {
            upstream_conf;
            allow 127.0.0.1;
            deny all;
        }
    }
}
```

> 为了传递配置命令给nginx，可以用任何方法发送一个HTTP请求，请求URI要包含`upstream_conf`指令，请求必须包含`upstream`参数设置服务器组的名字。例如，访问所有服务器组的备用服务器，就发送：`http://127.0.0.1/upstream_conf?stream=&upstream=appservers&backup=` 
添加一个新的服务器则发送：`http://127.0.0.1/upstream_conf?stream=&add=&upstream=appservers&server=appserv3.example.com:12345&weight=2&max_fails=3`   
删除一个服务器则发送：`http://127.0.0.1/upstream_conf?stream=&remove=&upstream=appservers&id=2`  
修改一个特定的服务器则发送逞id参数标识服务器和参数名：
`http://127.0.0.1/upstream_conf?stream=&upstream=appservers&id=2&down=`

**TCP负载均衡配置例子**

```nginx
stream {
    upstream stream_backend {
        least_conn;
        server backend1.example.com:12345 weight=5;
        server backend2.example.com:12345 max_fails=2 fail_timeout=30s;
        server backend3.example.com:12346 max_conns=3;
    }

    server {
        listen 12345;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }

    server {
        listen 12346;
        proxy_pass backend4.example.com:12346;
    }
}
```



### 使用代理协议
> 配置nginx,使nginx接收通过像HAproxy和Amazon Elastic负载均衡器等的代理服务器或负载均衡器传过来的客户端连接信息。

**概述**

> 代理协议会使nginx接收客户端连接信息并发送给代理服务器或负载均衡器如HAproxy和Amazon Elastic 均衡器。

**使用SSL，HTTP/2，SPDY和WebSocket的代理协议**

> 为了让nginx接受这些代理协议，则在**http**层级做如下修改：

1. 配置nginx接收代理协议头，给`listen`指令添加**proxy_protocol**参数：

    ```nginx
    server {
        listen 80   proxy_protocol;
        listen 443  ssl proxy_protocol;
        ...
    }
    ```

2. 在`set_real_ip_from`指令中指定IP地址或范围：

    ```nginx
    server {
        ...
        set_real_ip_from 192.168.1.0/24;
        ...
    }
    ```

3. 在`real_ip_header`指令中添加**proxy_protocol**参数，可以保持客户端的IP地址和端口号

    ```nginx
    server {
        ...
        real_ip_header proxy_protocol;
    }
    ```

4. 用`proxy_set_header`指令和`proxy_protocol_addr`变量传递客户端IP从nginx到上游服务器：

    ```nginx
    proxy_set_header X-Real-IP       $proxy_protocol_addr;
    proxy_set_header X-Forwarded-For $proxy_protocol_addr;
    ```

5. 添加`proxy_protocol_addr`变量到`log_format`指令里在http级别：

    ```nginx
    http {
        ...
        log_format combined '$proxy_protocol_addr - $remote_user [$time_local] '
                            '"$request" $status $body_bytes_sent '
                            '"$http_referer" "$http_user_agent"';
    }
    ```

**使用TCP流代理协议**

> nginx可以配置成传代理协议数据为TCP流。需要在stream级别的server块包含`proxy_protocol`指令：

```nginx
stream {
    server {
        listen 12345;
        proxy_pass example.com:12345;
        proxy_protocol on;
    }
}
```

**完整的例子**

```nginx
http {
    log_format combined '$proxy_protocol_addr - $remote_user [$time_local] '
                        '"$request" $status $body_bytes_sent '
                        '"$http_referer" "$http_user_agent"';
    ...

    server {
        server_name localhost;

        listen 80   proxy_protocol;
        listen 443  ssl proxy_protocol;

        ssl_certificate      /etc/nginx/ssl/public.example.com.pem;
        ssl_certificate_key  /etc/nginx/ssl/public.example.com.key;

        set_real_ip_from 192.168.1.0/24;
        real_ip_header   proxy_protocol;

        location /app/ {
            proxy_pass       http://backend1;
            proxy_set_header Host            $host;
            proxy_set_header X-Real-IP       $proxy_protocol_addr;
            proxy_set_header X-Forwarded-For $proxy_protocol_addr;
        }
    }
} 

stream {
...
    server {
        listen         12345;
        proxy_pass     example.com:12345;
        proxy_protocol on;
    }

}
```

***

## 安全

### 限制访问代理HTTP资源
> 控制基于客户端IP地址，HTTP身份验证的配置的访问，限制同时连接的数量，限制请求速度及每个连接的带宽。

**限制访问**

> 访问可以通过客户端IP或通过HTTP身份验证等被允许或拒绝。允许或拒绝来自相应IP的访问可以用`allow`和`deny`指令：

```nginx
location / {
    deny  192.168.1.2;
    allow 192.168.1.1/24;
    allow 127.0.0.1;
    deny  all;
}
```

> 开启身份验证，可以使用`auth_basic`指令。用户必须输入用户名和密码访问网站。用户名和密码必须放在`auth_basic_user_file`指令指定的文件中：

```nginx
server {
    ...
    auth_basic "closed website";
    auth_basic_user_file conf/htpasswd;
}
```

> 你可以指定网站的一些地方可以不需要验证即使整个网站是需要验证的。通过`auth_basic`指令的`off`参数可以在相应的块中取消对全局身份验证的继承，从而达到该块不需要验证的目的，如不允许访问整体网站但可以访问一些locations：

```nginx
server {
    ...
    auth_basic "closed website";
    auth_basic_user_file conf/htpasswd;

    location /public/ {
        auth_basic off;
    }
}
```

> 也可以两种策略结合起来做限制，使用`satisfy`指令即可，默认的可以设置成`all`参数，即客户需要同时满足IP地址及身份验证的合法性才能访问。当设置`any`参数是，即只要有一种条件满足即可访问：

```nginx
location / {
    satisfy any;

    allow 192.168.1.0/24;
    deny  all;

    auth_basic           "closed site";
    auth_basic_user_file conf/htpasswd;
}
```

**其他访问限制**

> 也可能会在以下方面做限制：

* 每个key-value的连接数(例如每个IP)
* 每个key-value的请求速率
* 一个连接的下载速度

> *注意*：IP是可以在网络设备上共享的，故要小心使用IP做限制

> *限制连接数*

> 可以用`limit_conn_zone`指令定义key和相应的缓存区域，第一个参数指定一个表达式做为key，第二个参数指定缓存区域的名字及大小：

```nginx
limit_conn_zone $binary_remote_address zone=addr:10m;
```

> 其次使用`limit_conn`指令可以应用限制到location或一个虚拟服务器server或整个http上下文。第一个参数指定共享的内存块名，第二个参数指定第个key允许的连接数：

```nginx
location /download/ {
        limit_conn addr 1;
}
```

> 这里用**$binary_remote_address**变量做为key来限制IP地址的连接数，而针对某个服务器的连接数限制可以用**$server_name**变量指定：

```nginx
http {
    limit_conn_zone $server_name zone=servers:10m;

    server {
        limit_conn servers 1000;
    }
}
```

> *限制请求速率*

> 用`limit_req_zone`指令来指定key和共享内存块来计数控制请求速率：

```nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
```

> `rate`参数指定了每秒或每分钟的请求数，一旦定义了共享内存块就可以用`limit_req`指令指定一个虚拟服务器或location的请求速率：

```nginx
location /search/ {
    limit_req zone=one burst=5;
}
```

> 如果超过了限制则会把多的请求放在等待队列里等待延迟处理，`burst`参数指定了最大的等待请求数，超过这个数的请求nginx会503错误响应。如果不希望延迟处理，则加`nodelay`参数：

```nginx
limit_req zone=one burst=5 nodelay;
```

> *限制带宽*

> 使用`limit_rate`指令来限制每个连接的带宽：

```nginx
location /download/ {
    limit_rate 50k;
}
```

> 有了这个设置，用户的每个连接的最大下载速度不能超过50 kilobytes，然而客户端可能会打开多个连接，如果要从整体上限制速度不超过指定的值，则连接数也应该进行限制：

```nginx
location /download/ {
    limit_conn addr 1;
    limit_rate 50k;
}
```

> 为了加强限制用户在下载一定量的数据之后的速率，可以用`limit_rate_after`指令，这个会比较合理让用户快速下载一定量的数据，而限制其余的数据：

```nginx
limit_rate_after 500k;
limit_rate 20k;
```

> 下面是一个完整的例子：

```nginx
http {
    limit_conn_zone $binary_remote_address zone=addr:10m

    server {
        root /www/data;
        limit_conn addr 5;

        location / {
        }

        location /download/ {
            limit_conn addr 1;
            limit_rate 1m;
            limit_rate 50k;
        }
    }
}
```

### 限制访问代理TCP资源
> 控制基于客户端IP地址的访问，限制同时连接数，限制每个连接的带宽。

**限制通过IP地址访问**

> 可以在`stream`上下文或`server`块中用`allow`和`deny`指令来限制客户端相应的IP地址访问：

```nginx
stream {
    ...
    server {
        listen 12345;
        deny   192.168.1.2;
        allow  192.168.1.1/24;
        allow  2001:0db8::/32;
        deny   all;
    }
}
```

> 规则会按顺序从上到下执行，如果第一个指令是`deny all`，则前面所有的`allow`指令会不起作用。

**限制TCP连接数**

> 你可以限制同时来自同一IP地址的TCP连接数，这针对阻止denial-of-service(Dos)攻击会很有用。首先让我们来定义存储到一个服务器的最大TCP连接的内存区域和连接相应的key，这用`limit_conn_zone`指令在`stream`上下文中指定：

```nginx
stream {
    ...
    limit_conn_zone $binary_remote_addr zone=ip_addr:10m;
    ...
}
```

> 定义了zone之后，用`limit_conn`指令来限制具体的连接，第一个参数指定先前定义的zone的名字，第二个参数指定每个IP允许连接的最大数，注意不能的设置可以共享IP地址：

```nginx
stream {
    ...
    limit_conn_zone $binary_remote_addr zone=ip_addr:10m;

    server {
        ...
        limit_conn ip_addr 1;
    }
}
```

**限制带宽**

> 你可以用`proxy_download_rate`和`proxy_upload_rate`指令配置下载和上传速度：

```nginx
server {
    ...
    proxy_download_rate 100k;
    proxy_upload_rate   50k;
}
```

> 同样客户端会有多个TCP连接，为了从整体上控制速度，则需要限制下连接数：

```nginx
stream {
    ...
    limit_conn_zone $binary_remote_addr zone=ip_addr:10m;

    server {
        ...
        limit_conn ip_addr 1;
        proxy_download_rate 100k;
        proxy_upload_rate   50k;
    }
}
```

### 日志和监控
> 记录错误和请求日志，监控nginx的状态和性能

**设置错误日志**

> 用`error_log`指令记录日志到特定文件、stderr、或系统日志中，并指定要记录的最小的消息级别，默认的错误日志放在logs/error.log：

```nginx
error_log logs/error.log warn;
```

> 上面情况是*warn,error,crit,alert,emerg*级别的日志都会打印。`error_log`指令可以放在**http**,**stream**,**server**和**location**层级。

**设置访问日志**

> nginx写客户端请求信息到访问日志中当请求被处理完时，默认情况下，访问日志在logs/access.log，且日志按事先定义好的格式打印。为了重写默认设置可以用`log_format`指令来配置，同时用`access_log`指令来指定日志文件的位置。下面的例子定义了日志格式继承之前的组合格式添加了压缩比率值：

```nginx
http {
    log_format compression '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                          '"$http_referer" "$http_user_agent" "$gzip_ratio"';

    server {
        gzip on;
        access_log /spool/logs/nginx-access.log compression;
        ...
    }
}
```

> 也可以记录不同的时间值来查找一些问题：

* 可以用`$upstream_connect_time`变量来记录与上游服务器建立连接花费的时间
* 用`$upstream_header_time`变量来记录在建立连接和收到上游服务器响应头的第一个字节之间的时间
* 用`$upstream_response_time`变量来记录在建立连接后和收到上游服务器所有响应后这之间的时间
* 用`$request_time`变量记录处理请求所花费的时间

> 所有的时间值都是以毫秒级为单位的：

```nginx
http {
    log_format upstream_time '$remote_addr - $remote_user [$time_local] '
                             '"$request" $status $body_bytes_sent '
                              '"$http_referer" "$http_user_agent"'
                               'rt=$request_time uct="$upstream_connect_time"'
                                'uht="$upstream_header_time"'
                                  'urt="$upstream_response_time"';

    server {
        access_log /spool/logs/nginx-access.log upstream_time;
        ...
    }
}
```

> 这里有怎么赢取结果时间值的规则：

* 当请求是通过多个服务器处理时，变量会包含以,号分隔的多个值
* 当有内部重定向时，时间值将通过分号分隔
* 当请求没有到达服务器或没有收到完整的头时，值为0
* 如果在连接上游服务器时发生内部错误或来自红缓存的响应，则值包含'-'

> 也可以开启缓存来优化日志，缓存经常使用的文件描述符，为了开启缓存，可以用`access_log`指令的buffer参数指定缓存的大小。当下一条日志放不到缓存里时，才会把缓存里的数据放在文件里。可以用`open_log_file_cache`指令开启日志文件描述符缓存。

*开启条件日志*

> 有条件的日记记录，不包括访问日志琐碎的或非重要的日志条目。在nginx中通过`access_log`指令的if参数开启条件日志：

```nginx
map $status $loggable {
    ~^[23]  0;
    default 1;
}

access_log /path/to/access.log combined if=$loggable;
```

**记录到系统日志**

> 系统日志是计算机消息记录标准，并允许收集单个系统日志服务器上的不同设备的日志信息。在nginx中记录日志到系统日志中可以通过`error_log`和`access_log`指令中的**syslog:*前缀来设置。系统日志可以记录到*server=*指定的一个域名或一个IP地址或unix套接字路径。可以指定域名或IP的端口，默认是514：

```nginx
error_log server=unix:/var/log/nginx.sock debug;
access_log syslog:server=[2001:db8::1]:1234,facility=local7,tag=nginx,severity=info;
```

> 例子中错误日志将debug级别的信息写到unix套接字路径中，而访问日志将被写到IP地址端口号指定的服务系统日志中。

> ** facility= **参数指定了日志记录程序的类型默认是*local7*，也可以是*auth,authpriv, daemon, cron, ftp, lpr, kern, mail, news, syslog, user, uucp, local0 … local7*.

> **tag=**参数将应用一个自定义的tag到系统日志信息中。

> **serverity=**参数设置了系统日志级别，可以是*debug, info, notice, warn, error (default), crit, alert, and emerg*. 

**实时监控**

> NGINX提供一个实时现场活动的监控界面，显示您的HTTP和TCP上游服务器的关键负载和性能指标以及其他数据，包括

* NGINX版本，运行时间，和标识信息
* 连接和请求的累计和当前数
* 每个状态池status_zone的请求和响应数
* 每个动态配置的服务器组中的每个服务器的请求和响应数，还有健康检查和正常运行时间的统计数据
* 每个服务器的统计信息，包括其当前状态和总价值
* 每个缓存区的状态

> 所有的统计都在nginx包里提供的status.html页上显示

*开启实时监控*

> 开启该功能，你需要指定location去检查带/status的URI的匹配，同时location下要包含status指令。要开启status.html页面，你需要指定一个或多个location设置他：

```nginx
server {
    listen 127.0.0.1;
    root /usr/share/nginx/html;

    location /status {
        status;
    }

    location = /status.html {
    }
}
```

> 然后访问accessing http://127.0.0.1/status.html查看其统计信息


### 高可用性的nginx
> 基于保持活动连接的高可用性nginx的配置

*.......待续........*
