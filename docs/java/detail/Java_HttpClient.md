## 一些术语

### HTTP Message

包括报头部分和可选实体。 有两种消息，请求和响应。 它们在第一行的格式上不同，但两者都可以有头字段和可选实体。

### Entity

是与HTTP消息一起发送的数据。 例如，响应可以包含您作为实体下载的页面或图像，或者请求可以包括您在Web表单中输入的参数。 HTTP消息的实体可以具有任意数据格式，通常在头字段中指定为MIME类型。

## 实例

### 简单的Get请求

```java
/**
 * 简单的Get请求
 * @throws IOException
 */
@Test
public void testGetMethod() throws IOException {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    HttpGet httpGet = new HttpGet("http://httpbin.org/");
    CloseableHttpResponse response = httpClient.execute(httpGet);
    try {
        System.out.println(response.getStatusLine());
        HttpEntity entity = response.getEntity();
        System.out.println(EntityUtils.toString(entity));
        EntityUtils.consume(entity);

    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        response.close();
    }
}
```

### Post请求

```java
/**
 * 通过{@code BasicNameValuePair,HttpPost,UrlEncodedFormEntity}发送post请求
 * @throws IOException
 */
@Test
public void testPostMethod() throws IOException {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    HttpPost httpPost = new HttpPost("http://httpbin.org/");
    List<NameValuePair> pairs = new ArrayList<NameValuePair>();
    pairs.add(new BasicNameValuePair("name", "bill"));
    pairs.add(new BasicNameValuePair("password", "111"));
    httpPost.setEntity(new UrlEncodedFormEntity(pairs));
    CloseableHttpResponse response = httpClient.execute(httpPost);
    try {
        System.out.println(response.getStatusLine());
        HttpEntity entity = response.getEntity();
        System.out.println(EntityUtils.toString(entity));
        EntityUtils.consume(entity);
    } finally {
        response.close();
    }
}
```

### 使用响应处理程序处理HTTP响应

使用响应处理程序处理HTTP响应。 这是执行HTTP请求和处理HTTP响应的建议方式。这种方法使呼叫者能够专注于消化HTTP响应的过程，并将系统资源释放的任务委托给HttpClient。使用HTTP响应处理程序保证底层HTTP连接在所有情况下都将自动释放回连接管理器

```java
/**
 * 使用响应处理程序处理HTTP响应。 这是执行HTTP请求和处理HTTP响应的建议方式。
 * 这种方法使呼叫者能够专注于消化HTTP响应的过程，并将系统资源释放的任务委托给HttpClient。
 * 使用HTTP响应处理程序保证底层HTTP连接在所有情况下都将自动释放回连接管理器
 * @throws IOException
 */
@Test
public void testResponseHandler() throws IOException {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    try {
        HttpGet httpGet = new HttpGet("http://httpbin.org/");
        System.out.println("executing request " + httpGet.getRequestLine());

        ResponseHandler<String> responseHandler = new ResponseHandler<String>() {
            @Override
            public String handleResponse(HttpResponse httpResponse) throws ClientProtocolException, IOException {
                int status = httpResponse.getStatusLine().getStatusCode();
                if (status >= 200 && status <= 300) {
                    HttpEntity entity = httpResponse.getEntity();
                    return entity != null ? EntityUtils.toString(entity) : null;
                } else {
                    throw new ClientProtocolException("unexpected response status: " + status);
                }
            }
        };
        String responseBody = httpClient.execute(httpGet, responseHandler);
        System.out.println("------------------------");
        System.out.println(responseBody);
    } finally {
        httpClient.close();
    }
}
```

### 手动释放连接资源

```java
/**
 * 手动释放连接资源
 * @throws Exception
 */
@Test
public void testManulConnectionRelease() throws Exception {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    try {
        HttpGet httpGet = new HttpGet("http://httpbin.org/get");
        System.out.println("executing request " + httpGet.getRequestLine());
        CloseableHttpResponse response = httpClient.execute(httpGet);
        try {
            System.out.println("---------------------");
            System.out.println(response.getStatusLine());
            HttpEntity entity = response.getEntity();

            //如果没有响应数据entity,则不需要释放连接
            if (entity != null) {
                InputStream inputStream = entity.getContent();
                try {
                    byte[] con = new byte[1024];
                    int w = 0;
                    while ((w = inputStream.read(con)) > 0) {
                        System.out.println(new String(con));
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    //关闭输入流会触发释放连接
                    inputStream.close();
                }
            }
        } finally {
            response.close();
        }
    } finally {
        httpClient.close();
    }
}
```

### 如何自定义和配置HTTP请求执行和连接管理的最常见的方面

```java
/**
 * 如何自定义和配置HTTP请求执行和连接管理的最常见的方面
 * @throws IOException
 */
@Test
public void testCoustomConfigruation() throws IOException {
    HttpMessageParserFactory<HttpResponse> responseParserFactory = new DefaultHttpResponseParserFactory() {
        @Override
        public HttpMessageParser<HttpResponse> create(SessionInputBuffer buffer, MessageConstraints constraints) {
            LineParser lineParser = new BasicLineParser() {
                @Override
                public Header parseHeader(CharArrayBuffer buffer) throws ParseException {
                    try {
                        return super.parseHeader(buffer);
                    } catch (ParseException e) {
                        return new BasicHeader(buffer.toString(), null);
                    }
                }
            };
            return new DefaultHttpResponseParser(
                    buffer, lineParser, DefaultHttpResponseFactory.INSTANCE, constraints) {

                @Override
                protected boolean reject(final CharArrayBuffer line, int count) {
                    // try to ignore all garbage preceding a status line infinitely
                    return false;
                }

            };
        }

    };

    HttpMessageWriterFactory<HttpRequest> requestWriterFactory = new DefaultHttpRequestWriterFactory();

    org.apache.http.conn.HttpConnectionFactory<HttpRoute, ManagedHttpClientConnection> connFactory = new ManagedHttpClientConnectionFactory(
            requestWriterFactory, responseParserFactory);

    SSLContext sslcontext = SSLContexts.createSystemDefault();

    Registry<ConnectionSocketFactory> socketFactoryRegistry = RegistryBuilder.<ConnectionSocketFactory>create()
            .register("http", PlainConnectionSocketFactory.INSTANCE)
            .register("https", new SSLConnectionSocketFactory(sslcontext))
            .build();

    // Use custom DNS resolver to override the system DNS resolution.
    DnsResolver dnsResolver = new SystemDefaultDnsResolver() {

        @Override
        public InetAddress[] resolve(final String host) throws UnknownHostException {
            if (host.equalsIgnoreCase("myhost")) {
                return new InetAddress[]{InetAddress.getByAddress(new byte[]{127, 0, 0, 1})};
            } else {
                return super.resolve(host);
            }
        }

    };

    // Create a connection manager with custom configuration.
    PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager(
            socketFactoryRegistry, connFactory, dnsResolver);

    // Create socket configuration
    SocketConfig socketConfig = SocketConfig.custom()
            .setTcpNoDelay(true)
            .build();
    // Configure the connection manager to use socket configuration either
    // by default or for a specific host.
    connManager.setDefaultSocketConfig(socketConfig);
    connManager.setSocketConfig(new HttpHost("somehost", 80), socketConfig);
    // Validate connections after 1 sec of inactivity
    connManager.setValidateAfterInactivity(1000);

    // Create message constraints
    MessageConstraints messageConstraints = MessageConstraints.custom()
            .setMaxHeaderCount(200)
            .setMaxLineLength(2000)
            .build();
    // Create connection configuration
    ConnectionConfig connectionConfig = ConnectionConfig.custom()
            .setMalformedInputAction(CodingErrorAction.IGNORE)
            .setUnmappableInputAction(CodingErrorAction.IGNORE)
            .setCharset(Consts.UTF_8)
            .setMessageConstraints(messageConstraints)
            .build();
    // Configure the connection manager to use connection configuration either
    // by default or for a specific host.
    connManager.setDefaultConnectionConfig(connectionConfig);
    connManager.setConnectionConfig(new HttpHost("somehost", 80), ConnectionConfig.DEFAULT);

    // Configure total max or per route limits for persistent connections
    // that can be kept in the pool or leased by the connection manager.
    connManager.setMaxTotal(100);
    connManager.setDefaultMaxPerRoute(10);
    connManager.setMaxPerRoute(new HttpRoute(new HttpHost("somehost", 80)), 20);

    // Use custom cookie store if necessary.
    CookieStore cookieStore = new BasicCookieStore();
    // Use custom credentials provider if necessary.
    CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
    // Create global request configuration
    RequestConfig defaultRequestConfig = RequestConfig.custom()
            .setCookieSpec(CookieSpecs.DEFAULT)
            .setExpectContinueEnabled(true)
            .setTargetPreferredAuthSchemes(Arrays.asList(AuthSchemes.NTLM, AuthSchemes.DIGEST))
            .setProxyPreferredAuthSchemes(Arrays.asList(AuthSchemes.BASIC))
            .build();

    // Create an HttpClient with the given custom dependencies and configuration.
    CloseableHttpClient httpclient = HttpClients.custom()
            .setConnectionManager(connManager)
            .setDefaultCookieStore(cookieStore)
            .setDefaultCredentialsProvider(credentialsProvider)
            .setProxy(new HttpHost("myproxy", 8080))
            .setDefaultRequestConfig(defaultRequestConfig)
            .build();

    try {
        HttpGet httpget = new HttpGet("http://httpbin.org/get");
        // Request configuration can be overridden at the request level.
        // They will take precedence over the one set at the client level.
        RequestConfig requestConfig = RequestConfig.copy(defaultRequestConfig)
                .setSocketTimeout(5000)
                .setConnectTimeout(5000)
                .setConnectionRequestTimeout(5000)
                .setProxy(new HttpHost("myotherproxy", 8080))
                .build();
        httpget.setConfig(requestConfig);

        // Execution context can be customized locally.
        HttpClientContext context = HttpClientContext.create();
        // Contextual attributes set the local context level will take
        // precedence over those set at the client level.
        context.setCookieStore(cookieStore);
        context.setCredentialsProvider(credentialsProvider);

        System.out.println("executing request " + httpget.getURI());
        CloseableHttpResponse response = httpclient.execute(httpget, context);
        try {
            System.out.println("----------------------------------------");
            System.out.println(response.getStatusLine());
            System.out.println(EntityUtils.toString(response.getEntity()));
            System.out.println("----------------------------------------");

            // Once the request has been executed the local context can
            // be used to examine updated state and various objects affected
            // by the request execution.

            // Last executed request
            context.getRequest();
            // Execution route
            context.getHttpRoute();
            // Target auth state
            context.getTargetAuthState();
            // Proxy auth state
            context.getTargetAuthState();
            // Cookie origin
            context.getCookieOrigin();
            // Cookie spec used
            context.getCookieSpec();
            // User security token
            context.getUserToken();

        } finally {
            response.close();
        }
    } finally {
        httpclient.close();
    }
}
```

### 通过HttpGet.abort()方法可以中途放弃请求

```java
/**
 * 通过HttpGet.abort()方法可以中途放弃请求
 *
 * @throws IOException
 */
@Test
public void testClientAbortMethod() throws IOException {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    try {
        HttpGet httpGet = new HttpGet("http://httpbin.org/get");
        System.out.println("Executing rquest " + httpGet.getURI());
        CloseableHttpResponse response = httpClient.execute(httpGet);
        try {
            System.out.println("------------------------");
            System.out.println(response.getStatusLine());
            if (response.getStatusLine().getStatusCode() == 200) {
                System.out.println(EntityUtils.toString(response.getEntity()));
            }
            EntityUtils.consume(response.getEntity());

            httpGet.abort();
        } finally {
            response.close();
        }
    } finally {
        httpClient.close();
    }
}
```

### 设置通过用户认证访问一个网站

```java
/**
 * 设置通过用户认证访问一个网站
 *
 * @throws IOException
 */
@Test
public void testClientAuthentication() throws IOException {
    CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
    credentialsProvider.setCredentials(new AuthScope("httpbin.org", 80), new UsernamePasswordCredentials("user", "passwd"));
    CloseableHttpClient httpClient = HttpClients.custom()
            .setDefaultCredentialsProvider(credentialsProvider)
            .build();
    try {
        HttpGet httpGet = new HttpGet("http://httpbin.org/basic-auth/user/passwd");
        System.out.println("Executing request " + httpGet.getRequestLine());

        CloseableHttpResponse response = httpClient.execute(httpGet);
        try {
            System.out.println("-------------------------------");
            System.out.println(response.getStatusLine());
            System.out.println(EntityUtils.toString(response.getEntity()));
        } finally {
            response.close();
        }
    } finally {
        httpClient.close();
    }
}
```

### 通过代理发送请求

```java
/**
 * 通过代理发送请求
 *
 * @throws IOException
 */
@Test
public void testClientExecuteProxy() throws IOException {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    try {
        HttpHost target = new HttpHost("httpbin.org", 443, "https");
        HttpHost proxy = new HttpHost("127.0.0.1", 8080, "http");
        RequestConfig requestConfig = RequestConfig.custom()
                .setProxy(proxy)
                .build();
        HttpGet request = new HttpGet("/");
        request.setConfig(requestConfig);

        System.out.println("executing request " + request.getRequestLine() + " to " + target + " via " + proxy);

        CloseableHttpResponse response = httpClient.execute(target, request);
        try {
            System.out.println("-------------------------");
            System.out.println(response.getStatusLine());
            System.out.println(EntityUtils.toString(response.getEntity()));
        } finally {
            response.close();
        }
    } finally {
        httpClient.close();
    }
}
```

### 一个简单的示例，显示通过认证代理通过安全连接执行HTTP请求

```java
/**
 * 一个简单的示例，显示通过认证代理通过安全连接执行HTTP请求。
 */
@Test
public void testClientProxyAuthentication() throws IOException {
    CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
    credentialsProvider.setCredentials(new AuthScope("localhost", 8888), new UsernamePasswordCredentials("squid", "squid"));
    credentialsProvider.setCredentials(new AuthScope("httpbin.org", 80), new UsernamePasswordCredentials("user", "passwd"));

    CloseableHttpClient httpClient = HttpClients.custom()
            .setDefaultCredentialsProvider(credentialsProvider)
            .build();

    try {
        HttpHost target = new HttpHost("httpbin.org", 80, "http");
        HttpHost proxy = new HttpHost("localhost", 8888);

        RequestConfig config = RequestConfig.custom()
                .setProxy(proxy)
                .build();

        HttpGet httpGet = new HttpGet("/basic-auth/user/passwd");
        httpGet.setConfig(config);

        System.out.println("executing request " + httpGet.getRequestLine() + " to " + target + " via " + proxy);

        CloseableHttpResponse response = httpClient.execute(target, httpGet);

        try {
            System.out.println("_____________________________");
            System.out.println(response.getStatusLine());
            System.out.println(EntityUtils.toString(response.getEntity()));
        } finally {
            response.close();
        }
    } finally {
        httpClient.close();

    }
}
```

### 示例如何使用无缓冲区块编码的POST请求。

```java
/**
 * 示例如何使用无缓冲区块编码的POST请求。
 */
@Test
public void testClientChunkEncodedPost() throws Exception {
    String file_str = "/Users/zhangbingbing/tmp.log";

    CloseableHttpClient httpClient = HttpClients.createDefault();
    try {
        HttpPost httpPost = new HttpPost("http://httpbin.org/post");
        File file = new File(file_str);
        InputStreamEntity reqEntity = new InputStreamEntity(new FileInputStream(file), -1, ContentType.APPLICATION_OCTET_STREAM);
        reqEntity.setChunked(true);

        httpPost.setEntity(reqEntity);

        System.out.println("executing request: " + httpPost.getRequestLine());

        CloseableHttpResponse response = httpClient.execute(httpPost);
        try {
            System.out.println("----------------------------------");
            System.out.println(response.getStatusLine());
            System.out.println(EntityUtils.toString(response.getEntity()));
        } finally {
            response.close();
        }
    } finally {
        httpClient.close();
    }
}
```

### 使用自定义属性填充的本地HTTP上下文

```java
/**
 * 使用自定义属性填充的本地HTTP上下文
 */
@Test
public void testClientCustomContext() throws Exception {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    try {
        //创建本地cookie存储实例
        CookieStore cookieStroe = new BasicCookieStore();

        //创建本地http上下文
        HttpClientContext localContext = HttpClientContext.create();

        //绑定cookie到http上下文
        localContext.setCookieStore(cookieStroe);

        HttpGet httpget = new HttpGet("http://httpbin.org/cookies");
        System.out.println("executing request " + httpget.getRequestLine());

        //把本地上下文作为参数传递
        CloseableHttpResponse response = httpClient.execute(httpget, localContext);
        try {
            System.out.println("-----------------------");
            System.out.println(response.getStatusLine());
            List<Cookie> cookies = cookieStroe.getCookies();
            for (int i = 0; i < cookies.size(); i++) {
                System.out.println("Local cookie: " + cookies.get(i));
            }
            System.out.println(EntityUtils.toString(response.getEntity()));
            EntityUtils.consume(response.getEntity());

        } finally {
            response.close();
        }
    } finally {
        httpClient.close();
    }
}
```

### 使用HttpClient API执行基于表单的登录

```java
/**
 * 使用HttpClient API执行基于表单的登录
 */
@Test
public void testClientFormLogin() throws Exception {
    BasicCookieStore cookieStore = new BasicCookieStore();
    CloseableHttpClient httpClient = HttpClients.custom()
            .setDefaultCookieStore(cookieStore)
            .build();
    try {
        HttpGet httpGet = new HttpGet("https://someportal/");
        CloseableHttpResponse response = httpClient.execute(httpGet);
        try {
            HttpEntity entity = response.getEntity();
            System.out.println("login form get: " + response.getStatusLine());
            EntityUtils.consume(entity);

            System.out.println("Initial set of cookies:");
            List<Cookie> cookies = cookieStore.getCookies();
            if (cookies.isEmpty()) {
                System.out.println("None");
            } else {
                for (int i = 0; i < cookies.size(); i++) {
                    System.out.println("- " + cookies.get(i).toString());
                }
            }
        } finally {
            response.close();
        }


        HttpUriRequest login = RequestBuilder.post()
                .setUri(new URI("https://someportal/"))
                .addParameter("IDToken1", "username")
                .addParameter("IDToken2", "password")
                .build();
        CloseableHttpResponse response1 = httpClient.execute(login);
        try {
            HttpEntity entity = response1.getEntity();
            System.out.println("login form get: " + response1.getStatusLine());
            EntityUtils.consume(entity);

            System.out.println("post logon cookies:");
            List<Cookie> cookies = cookieStore.getCookies();
            if (cookies.isEmpty()) {
                System.out.println("none");
            } else {
                for (int i = 0; i < cookies.size(); i++) {
                    System.out.println("- " + cookies.get(i).toString());
                }
            }
        } finally {
            response1.close();
        }

    } finally {
        httpClient.close();
    }


}

```

### 多个线程执行GET请求

```java
/**
 * 多个线程执行GET请求
 */
@Test
public void testClientMultiThreadedExecution() throws Exception {
    //创建一个带ThreadSafeClientConnManager的httpclient，如果有多个线程使用httpclient就会用到连接管理器
    PoolingHttpClientConnectionManager manager = new PoolingHttpClientConnectionManager();
    manager.setMaxTotal(100);

    CloseableHttpClient httpClient = HttpClients.custom()
            .setConnectionManager(manager)
            .build();

    try {
        //创建多个请求URI
        String[] urisToGet = {
                "http://hc.apache.org/",
                "http://hc.apache.org/httpcomponents-core-ga/",
                "http://hc.apache.org/httpcomponents-client-ga/",
        };
        //为每个URI创建一个线程
        GetThread[] threads = new GetThread[urisToGet.length];
        for (int i = 0; i < threads.length; i++) {
            HttpGet httpGet = new HttpGet(urisToGet[i]);
            threads[i] = new GetThread(httpClient, httpGet, i);
        }
        //启动线程
        for (int j = 0; j < threads.length; j++) {
            threads[j].start();
        }
        //等待线程都运行完毕
        for (int j = 0; j < threads.length; j++) {
            threads[j].join();
        }
    } finally {
        httpClient.close();
    }


}

/**
 * 执行Get请求的线程
 */
static class GetThread extends Thread {
    private final CloseableHttpClient httpClient;
    private final HttpContext context;
    private final HttpGet httpGet;
    private final int id;

    public GetThread(CloseableHttpClient httpClient, HttpGet httpGet, int id) {
        this.httpClient = httpClient;
        this.context = new BasicHttpContext();
        this.httpGet = httpGet;
        this.id = id;
    }

    /**
     * 执行GetMethod方法打印一些状态信息
     */
    @Override
    public void run() {
        try {
            System.out.println(id + " - about to get something from " + httpGet.getURI());
            CloseableHttpResponse response = this.httpClient.execute(this.httpGet, this.context);
            try {
                System.out.println(id + " - get executed");
                HttpEntity entity = response.getEntity();
                if (entity != null) {
                    byte[] bytes = EntityUtils.toByteArray(entity);
                    System.out.println(id + " - " + bytes.length + " bytes read");
                }
            } finally {
                response.close();
            }
        } catch (Exception e) {
            System.out.println(id + " - error: " + e);
        }
    }
}
```

### 使用自定义SSL上下文创建安全连接

```java
/**
 * 使用自定义SSL上下文创建安全连接
 */
@Test
public void testClientCustomSSL() throws Exception {
    SSLContext sslContext = SSLContexts.custom()
            .loadTrustMaterial(new File("my.keystore"), "nopassword".toCharArray(), new TrustSelfSignedStrategy())
            .build();

    SSLConnectionSocketFactory sslConnectionSocketFactory = new SSLConnectionSocketFactory(sslContext, new String[]{"TLSv1"}, null, SSLConnectionSocketFactory.getDefaultHostnameVerifier());

    CloseableHttpClient httpClient = HttpClients.custom()
            .setSSLSocketFactory(sslConnectionSocketFactory)
            .build();

    try {

        HttpGet httpget = new HttpGet("https://httpbin.org/");

        System.out.println("Executing request " + httpget.getRequestLine());

        CloseableHttpResponse response = httpClient.execute(httpget);
        try {
            HttpEntity entity = response.getEntity();

            System.out.println("----------------------------------------");
            System.out.println(response.getStatusLine());
            EntityUtils.consume(entity);
        } finally {
            response.close();
        }
    } finally {
        httpClient.close();
    }
}

### 显示如何可以自定义HttpClient以使用BASIC方案抢占式进行身份验证。 通常，抢先认证可以被认为不如对认证质询的响应安全，因此不鼓励

```java
/**
 * 显示如何可以自定义HttpClient以使用BASIC方案抢占式进行身份验证。 通常，抢先认证可以被认为不如对认证质询的响应安全，因此不鼓励。
 * @throws Exception
 */
@Test
public void testClientPreemptiveBasicAuthentication() throws Exception {
    HttpHost target = new HttpHost("httpbin.org", 80, "http");
    CredentialsProvider credsProvider = new BasicCredentialsProvider();
    credsProvider.setCredentials(
            new AuthScope(target.getHostName(), target.getPort()),
            new UsernamePasswordCredentials("user", "passwd"));
    CloseableHttpClient httpclient = HttpClients.custom()
            .setDefaultCredentialsProvider(credsProvider).build();
    try {

        // Create AuthCache instance
        AuthCache authCache = new BasicAuthCache();
        // Generate BASIC scheme object and add it to the local
        // auth cache
        BasicScheme basicAuth = new BasicScheme();
        authCache.put(target, basicAuth);

        // Add AuthCache to the execution context
        HttpClientContext localContext = HttpClientContext.create();
        localContext.setAuthCache(authCache);

        HttpGet httpget = new HttpGet("http://httpbin.org/hidden-basic-auth/user/passwd");

        System.out.println("Executing request " + httpget.getRequestLine() + " to target " + target);
        for (int i = 0; i < 3; i++) {
            CloseableHttpResponse response = httpclient.execute(target, httpget, localContext);
            try {
                System.out.println("----------------------------------------");
                System.out.println(response.getStatusLine());
                System.out.println(EntityUtils.toString(response.getEntity()));
            } finally {
                response.close();
            }
        }
    } finally {
        httpclient.close();
    }
}
```

### 如何可以自定义HttpClient以使用DIGEST方案抢占式进行身份验证。 通常，抢先认证可以被认为不如对认证质询的响应安全，因此不鼓励

```java
@Test
public void testClientPreemptiveDigestAuthentication() throws Exception {
    HttpHost target = new HttpHost("httpbin.org", 80, "http");
    CredentialsProvider credsProvider = new BasicCredentialsProvider();
    credsProvider.setCredentials(
            new AuthScope(target.getHostName(), target.getPort()),
            new UsernamePasswordCredentials("user", "passwd"));
    CloseableHttpClient httpclient = HttpClients.custom()
            .setDefaultCredentialsProvider(credsProvider)
            .build();
    try {

        // Create AuthCache instance
        AuthCache authCache = new BasicAuthCache();
        // Generate DIGEST scheme object, initialize it and add it to the local
        // auth cache
        DigestScheme digestAuth = new DigestScheme();
        // Suppose we already know the realm name
        digestAuth.overrideParamter("realm", "some realm");
        // Suppose we already know the expected nonce value
        digestAuth.overrideParamter("nonce", "whatever");
        authCache.put(target, digestAuth);

        // Add AuthCache to the execution context
        HttpClientContext localContext = HttpClientContext.create();
        localContext.setAuthCache(authCache);

        HttpGet httpget = new HttpGet("http://httpbin.org/digest-auth/auth/user/passwd");

        System.out.println("Executing request " + httpget.getRequestLine() + " to target " + target);
        for (int i = 0; i < 3; i++) {
            CloseableHttpResponse response = httpclient.execute(target, httpget, localContext);
            try {
                System.out.println("----------------------------------------");
                System.out.println(response.getStatusLine());
                System.out.println(EntityUtils.toString(response.getEntity()));
            } finally {
                response.close();
            }
        }
    } finally {
        httpclient.close();
    }
}
```

### 使用multipart / form编码的POST请求

```java

/**
 * 示例如何使用multipart / form编码的POST请求
 */
@Test
public void testClientMultipartFormPost() throws Exception{
    String fileStr = "/Users/work/tmp.log";
    CloseableHttpClient httpClient = HttpClients.createDefault();

    try {
        HttpPost httpPost = new HttpPost("http://localhost:8080/servlets-examples/servlet/RequestInfoExample");
        FileBody bin = new FileBody(new File(fileStr));
        StringBody comment = new StringBody("a binary file of some kind", ContentType.TEXT_PLAIN);
        HttpEntity httpEntity = MultipartEntityBuilder.create()
                .addPart("bin", bin)
                .addPart("comment",comment)
                .build();
        httpPost.setEntity(httpEntity);

        System.out.println("executing request " + httpPost.getRequestLine());
        CloseableHttpResponse response = httpClient.execute(httpPost);
        try {
            System.out.println("----------------------------------------");
            System.out.println(response.getStatusLine());
            HttpEntity resEntity = response.getEntity();
            if (resEntity != null) {
                System.out.println("Response content length: " + resEntity.getContentLength());
            }
            EntityUtils.consume(resEntity);
        } finally {
            response.close();
        }
    }finally {
        httpClient.close();
    }
}
```

