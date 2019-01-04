## 协议栈
* HTTP                          _应用层_
* TSL or SSL                    _安全层_
* TCP                           _传输层_
* IP                            _网络层_
* 网络接口                      _数据链路层_

![aa](https://github.com/bingbo/blog/blob/master/images/clipboard.png)

## 网络接口

**`socket.socket(socket.AF_INET, socket.SOCK_STREAM)`**
> **地址协议家族，用于第一个参数**

* `socket.AF_UNIX`
* `socket.AF_INET`
* `socket.AF_INET6`

> **套接字类型，如TCP、UDP等，用于第二个参数**

1. `socket.SOCK_STREAM`
2. `socket.SOCK_DGRAM`
3. `socket.SOCK_RAW`
4. `socket.SOCK_RDM`
5. `socket.SOCK_SEQPACKET`

> **调用**

应用层接口->linux用户层接口->linux内核系统层接口




## HTTP协议
### HTTP报文
是在HTTP应用程序之间发送的数据块。这些数据块以一些文本形式的元信息开关，这些信息描述了报文的内容及含义，后面跟着可选的数据部分。这些报文在客户端、服务器和代理之间流动。
> **报文的组成**

* 对报文进行描述的起始行
* 包含属性的首部块
* 以及可选的、包含数据的主体部分

_每行由两个字符组成的行终止序列作为结束，其中包括一个回车符和一个换行符。_
> **请求报文格式**

` <method> <request-url> <version>`

` <headers>`


` <entity-body>`

> **响应报文格式**

`<version> <status> <reason-phrase>`

`<headers>`


`<entity-body>`
> **请求方法**


方法 | 描述 | 是否包含主体
---- | --- | -------------
GET | 从服务器获取一份文档 | 否
HEAD | 只从服务器获取文档的首部 | 否
POST | 向服务器发送需要处理的数据 | 是
PUT | 将请求的主体部分存储在服务器上 | 是
TRACE | 对可能经过代理服务器传送到服务器上去的报文进行追踪 | 否
OPTIONS | 决定可以在服务器上执行哪些方法 | 否
DELETE | 从服务器上删除一份文档 | 否


> **状态码**

<table>
<tr><th>状态码</th><th>原因短语</th><th>含义</th></tr>
<tr><td>200	</td><td>OK	</td><td>请求没问题，实体的主体部分包含了所请求的资源</td></tr>
<tr><td>201	</td><td>Created	</td><td>用于创建服务器对象的请求(比如，PUT),响应的实体主体部分中应该包含各种引用了已创建的资源的URL，location首部包含的则是最具体的引用，服务器必须在发送这个状态码之前创建好对象</td></tr>
<tr><td>202	</td><td>Accepted	</td><td>请求已被接受，但服务器还未对其执行任何动作。不能保证 服务器会完成请求；这只是意味着接受请求时，它看起来是有效的。服务器应该在实体的主体部分包含对请求状态的描述，或许还应该有对请求完成时间的估计</td></tr>
<tr><td>203	</td><td>Non-Authoritative Information	</td><td>实体首部包含的信息不是来自于源端服务器，而是来自资源的一份副本</td></tr>
<tr><td>204	</td><td>No Content	</td><td>响应报文中包含若干首部和一个状态行，但没有实体的主体部分。主要用于在浏览器不转为显示新文档的情况下，对其进行更新</td></tr>
<tr><td>205	</td><td>Reset Content	</td><td>另一个主要用于浏览器的代码，负责告知浏览器清除当前页面中的所有HTML表单元素</td></tr>
<tr><td>206	</td><td>Partial Content	</td><td>成功执行了一个部分或Range（范围)请求。206响应中必须包含Content-Range、Date惟及ETag或Content-Loaction首部</td></tr>
<tr><td>300	</td><td>Multiple Choices	</td><td>客户端请求一个实际指向多个资源的URL时会返回这个状态码，比如服务器上有某个HTML文档的英语和法语版本。返回这个代码时会带有一个选项列表；这样用户就可以选择他希望使用的那一项了</td></tr>
<tr><td>301	</td><td>Moved Permanently	</td><td>在请求的URL已被移除时使用。响应的Location首部中应该包含资源现在所处的URL</td></tr>
<tr><td>302	</td><td>Found	</td><td>与301状态码类似，但是，客户端应该使用Location首部给出的URL来临时写信资源</td></tr>
<tr><td>303	</td><td>See Other	</td><td>告知客户端应该用另一个URL来获取资源。新的URL位于响应报文的Location首部。其主要目的是允许POST请求的响应将客户端定向到某个资源上去</td></tr>
<tr><td>304	</td><td>Not Modified	</td><td>客户端可以通过所包含的请求首部，使其请求变成有条件的。如果客户端发起了一个投机倒把GET请求，而最近资源未被修改的话，就可以用这个状态码来说明资源未被悠。带有这个状态码的响应不应该包含实体的主体部分</td></tr>
<tr><td>305	</td><td>Use Proxy	</td><td>用来说明必须通过一个代码来访问资源；代理的位置由Location首部给出，很重要的一点是，客户端是相对某个特定资源来解析这条响应的，不能假定所有请求，甚至所有对持有所请求资源的服务器的请求都通过这个代理 进行。如果客户端错误地让代理介入了某条请求，可能会引发破坏性的行为，而且会造成安全漏洞
</td></tr><tr><td>306	</td><td>未使用	</td><td>当前未使用
</td></tr><tr><td>307	</td><td>Temporary Redirect	</td><td>与301状态码类似，但客户端应该wgetLocation首部给出的URL来临时定位资源。
</td></tr><tr><td>400	</td><td>Bad Request	</td><td>用于告知客户端它发送了 个错误的请求
</td></tr><tr><td>401	</td><td>Unauthorized	</td><td>与适当的首部一同返回，丰这些首部中请求客户端在获取对资源的访问权之前，对自己进行谁。
</td></tr><tr><td>402	</td><td>Payment Required	</td><td>现在这个状态码还未使用，但已经被 保留，以作未来之用
</td></tr><tr><td>403	</td><td>Forbidden	</td><td>用于说明请求被 服务器拒绝了。如果服务器想说明为什么拒绝请求，可以包含实体的主体部分来对原因进行描述。但这个状态码通常是在服务器不想说明拒绝原因的时候使用的
</td></tr><tr><td>404	</td><td>Not Found	</td><td>用于说明服务器无法找到所请求的URL，通常会包含一个实体，以便 客户端应用程序显示给用户看
</td></tr><tr><td>405	</td><td>Method Not allowed	</td><td>发起的请求中带有所请求的URL不支持的访求时，使用此状态码。应该在响应中包含Allow首部，以告知客户端对所请求的资源可以使用哪些方法 
</td></tr><tr><td>406	</td><td>Not Acceptable</td><td>	客户端可以指定参数来说明它们愿意接收什么类型的实体。服务器没有与客户端可以接受的URL相匹配的资源时，使用此代码。通常服务器会包含一些首部，以便客户端弄清楚为佳私语示无法满足
</td></tr><tr><td>407	</td><td>Proxy Authentication Required	</td><td>与401状态码类似，但用于要求对奖行谁的代理服务器
</td></tr><tr><td>408	</td><td>Request Timeout	</td><td>如果客户端完成请求所花的时间太长，服务器可以回阖家此状态码并关闭连接，超时时长随服务器的不同有所不同，但通常对所有的合法请求来说，都是够长的
</td></tr><tr><td>409	</td><td>Conflict	</td><td>用于说明请求可能在资源上引发的一些冲突。服务器担心请求会此发冲突时，可以发送此状态码。响应中应该主体
</td></tr><tr><td>410	</td><td>Gone	</td><td>与404类似，只是服务器曾经拥有过此资源。主要用于web商战的维护，这样服务器的管理者就可以在资源被移除的情况下通知客户端了
</td></tr><tr><td>411	</td><td>Length Required	</td><td>服务器要求在请求报文 中包含COntent-Length首部时使用
</td></tr><tr><td>412	</td><td>Precondition Failed	</td><td>客户端发起了条件请求，且其中一个条件失败了的时候使用。客户端包含了Expect首部时发起的就是条件请求。
</td></tr><tr><td>413	</td><td>Request Entity Too Large	</td><td>客户端发送的实体主体部分比服务器能够或者希望处理的要大时，使用此状态码
</td></tr><tr><td>414	</td><td>Request URI Too Large	</td><td>客户端所发请求中的请求URL比服务器能够或者希望处理的要长时，使用此状态码
</td></tr><tr><td>415	</td><td>Unsupported Media Type	</td><td>服务器无法理解或无法支持客户端所发实体的内容类型时，使用些状态码
</td></tr><tr><td>416	</td><td>Requested Range Not Satisfiable	</td><td>请求报文所请求的是指定资源的某个范围，而引范围无效或无法满足时，使用此状态码
</td></tr><tr><td>417	</td><td>Expectation Failed	</td><td>请求的Expect请求首部包含了一个期望，但服务器无法满足此期望时，使用此状态码，如果代理或其他中间应用程序有确切证据说明源端服务器会为某请求产生一个失败的期望，就可以 发送这个响应状态码
</td></tr><tr><td>500	</td><td>Internal Server Error	</td><td>服务器遇到一个妨碍它为请求提供服务时的错误时，使用此状态码
</td></tr><tr><td>501	</td><td>Not Implemented	</td><td>客户端发起的请求超出服务器的能力范围(比如，使用了服务器不支持的请求方法)时，使用此状态码
</td></tr><tr><td>502	</td><td>Bad Gateway	</td><td>作为代理或网关使用的服务器从请求响应链的下一条链路上收到了一条伪响应(比如，它无法连接到其父网关)时，使用此状态码
</td></tr><tr><td>503	</td><td>Service Unavailable	</td><td>用来说明服务器现在无法为请求提供服务，但将来可以。如果服务器知道什么时候资源会变为可用的，可以在响应中包含一个retry-after首部。
</td></tr><tr><td>504	</td><td>Gateway Timeout	</td><td>与状态码408类似，只是这里的响应来自一个网关或代理，它们在等待一个服务器对其请求进行响应时超时了
</td></tr><tr><td>505	</td><td>HTTP Version Not Supported	</td><td>服务器收到 的请求使用了它无法或不愿支持的版本时，使用此状态码。</td></tr>
</table>
	
### HTTP首部

> **通用首部** 

<table>
<tr><th>首部</th><th>描述</th><tr>
<tr><td>Connection</td><td>允许客户端和服务器指定与请求、响应连接有关的选项</td></tr>
<tr><td>Date</td><td>
提供日期和时间标志，说明报文是什么时间创建的</td></tr>
<tr><td>MIME-Version</td><td>
给出了发送端使用的MIME版本</td></tr>
<tr><td>Trailer</td><td>
如果报文 采用了分块传输编码(chunked transfer encoding)方式，就可以用这个首部列出位于报文拖挂(trailer)部分的首部集合</td></tr>
<tr><td>Transfer-Encoding</td><td>
告知接收端为了保证 的可靠传输，对报文采用了什么编码方式</td></tr>
<tr><td>Update</td><td>
给出了发送端可能想要“升级”使用的新版本或协议</td></tr>
<tr><td>Via</td><td>
显示了报文经过的中间节点(代理、网关)</td></tr>
<tr><td>Cache-Control</td><td>
用于随报文传送缓存指示</td></tr>
<tr><td>Pragma</td><td>
另一种随报文传送指示的方式，但并不专用于缓存</td></tr>
</table>
	
> **请求首部** 

<table>
<tr><th>首部</th><th>描述</th><tr>
<tr><td>Client-IP</td><td>
提供了运行客户端的机器的IP地址</td></tr>
<tr><td>From</td><td>
提供了客户端用户的E-mail地址</td></tr>
<tr><td>Host</td><td>
给出了接收请求的服务器的主机名和端口号</td></tr>
<tr><td>Referer</td><td>
提供了包含当前请求URI的文档的URL</td></tr>
<tr><td>UA-Color</td><td>
提供了与客户端显示器的显示颜色有关的信息</td></tr>
<tr><td>UA-CPU</td><td>
给出了客户端CPU的类型或制造商</td></tr>
<tr><td>UA-Disp</td><td>
提供了与客户端显示器能力有关的信息</td></tr>
<tr><td>UA-OS</td><td>
给出了运行在客户端机器上的操作系统名称及版本</td></tr>
<tr><td>UA-Pixels</td><td>
提供了客户端显示器的像素信息</td></tr>
<tr><td>User-Agent</td><td>
将发起请求的应用程序名称告知服务吕</td></tr>
<tr><td>
Accept首部</td>
</tr>
<tr><td>Accept</td><td>
告诉服务器能够发送哪些媒体类型</td></tr>
<tr><td>Accept-Charset</td><td>
告诉 服务器能够发送哪些编码方式</td></tr>
<tr><td>Accept-Language</td><td>
告诉服务器能够发送哪些语言</td></tr>
<tr><td>TE</td><td>
告诉服务器可以使用哪些扩展传输编码</td></tr>

<tr><td>
条件请求首部</td>
</tr>
<tr><td>Expect	</td><td>
允许客户端列出某请求所要求的服务器行为 </td></tr>
<tr><td>If-Match</td><td>
如果实体标记与文档当前的实体标记相匹配，就获取这份文档 </td></tr>
<tr><td>If-Modified-Since</td><td>
除非在某个指定的日期之后资源被 修改过，否则就限制这个请求</td></tr>
<tr><td>If-None-Match</td><td>
如果提供的实体标记与当前文档的实体标记不相符，就获取文档</td></tr>
<tr><td>If-Range</td><td>
允许对文档的某个范围进行条件请求</td></tr>
<tr><td>If-Unmodified-Since</td><td>
除非在某个指定日期之后资源没有被修改过，否则就限制这个请求</td></tr>
<tr><td>Range</td><td>
如果服务器支持范围请求，就请求资源的指定范围 </td></tr>

<tr><td>
安全请求首部</td>	
</tr>
<tr><td>Authorization</td><td>
包含了客户端提供给服务器，以便对其自身进行谁的数据</td></tr>
<tr><td>Cookie</td><td>
客户端用它向服务器传送一个令牌－－它并不是真正的安全首部，但确实隐含了安全功能</td></tr>
<tr><td>Cookie2</td><td>
用来说明请求端支持的cookie版本</td></tr>

<tr>
<td>
代理请求首部
</td>
</tr>

<tr><td>Max-Forward</td><td>
在通往源端服务器的路径上，将请求转发给其他代理或网关的最大次数</td></tr>
<tr><td>Proxy-Authorization</td><td>
与Authorizaiton首部相同，但这个首部是在与代理进行认证时使用的</td></tr>
<tr><td>Proxy-Connection</td><td>
与connection首部相同，但这个首部是在与代理建立连接时使用的</td></tr>


</table>

> 响应首部

<table>

<tr><th>首部</th><th>描述</th><tr>
<tr><td>Age</td><td>
响应持续时间</td></tr>
<tr><td>Public</td><td>
服务器为其资源支持的请求方法列表</td></tr>
<tr><td>Retry-After</td><td>
如果资源不可用的话，在此日期或时间重试</td></tr>
<tr><td>Server</td><td>
服务器应用程序软件的名称和版本</td></tr>
<tr><td>Title</td><td>
对HTML文档来说，就是HTML文档的源端给出的标题</td></tr>
<tr><td>Warning</td><td>
比原因短语中更详细的一些警告报文</td></tr>
<tr><td>协商首部</td></tr>

<tr><td>Accept-Range</td><td>
对此资源来说，服务器可接受的范围类型</td></tr>
<tr><td>Vary</td><td>
服务器查看的其他首部的列表，可能会使响应发生变化，也就是说，这是一个首部列表，服务器会根据这些首部的内容挑选出最适合的资源版本发送给客户端</td></tr>
<tr><td>安全响应首部	</td></tr>

<tr><td>Proxy-Authenticate</td><td>
来自代理的对客户端的质询列表</td></tr>
<tr><td>Set-Cookie</td><td>
不是真正的安全首部，但隐含有安全功能，可以在客户端设置一个令牌，以便服务器对客户端进行标识</td></tr>
<tr><td>Set-Cookie2</td><td>
与Set-Cookie</td></tr>
<tr><td>WWW-Authenticate</td><td>
来自服务器对客户端的质询列表
</td></tr>
</table>

> 实体首部

<table>
<tr><th>首部</th><th>描述</th><tr>
<tr><td>Allow</td><td>
列出了可以对此实体执行的请求方法L</td></tr>
<tr><td>Location</td><td>
告知客户端实体实际上位于何处，用于将接收端定向到资源的位置上去</td></tr>
<tr><td>内容首部</td></tr>

<tr><td>Content-Base</td><td>
解析主体中的相对URL时使用的基础URL</td></tr>
<tr><td>Content-Encoding</td><td>
对主体执行的任意编码方式</td></tr>
<tr><td>Content-Language</td><td>
理解主体时最适宜使用的自然语言</td></tr>
<tr><td>Content-Length</td><td>
主体的长度或尺寸</td></tr>
<tr><td>Content-Location</td><td>
资源实际所处的位置</td></tr>
<tr><td>Content-MD5</td><td>
主体的MD%校验和</td></tr>
<tr><td>Content-Range</td><td>
在整个资源中此实体表示的字节范围</td></tr>
<tr><td>Content-Type</td><td>
这个主体的对象类型</td></tr>
<tr><td>实体缓存首部</td></tr>

<tr><td>ETag</td><td>
与此实体相关的实体标记</td></tr>
<tr><td>Expires</td><td>
实体不再有效，要从原始的源端再次获取此实体的日期和时间</td></tr>
<tr><td>Last-Modified</td><td>
这个实体最后一次被 修改的日期和时间</td></tr>



</table>

> **Cache-Control首部的指令**

<table>

<tr><th>指令</th><th>报文类型</th><th>描述</th></tr>
<tr><td>no-cache</td><td>
请求
</td><td>在重新向服务器验证之前，不要返回文档的缓存副本</td></tr>
<tr><td>no-store</td><td>
请求
</td><td>不要返回文档的缓存副本。不要保存服务器的响应</td></tr>
<tr><td>max-age</td><td>
请求
</td><td>红艳艳中的文档不能超过指定的使用期</td></tr>
<tr><td>max-stale</td><td>
请求
</td><td>文档允许过期，但不能超过指令中指定的过期值</td></tr>
<tr><td>min-fresh</td><td>
请求
</td><td>文档的使用期不能小于这个指定的时间与它的当前戚时间之和。换句话说，响应必须至少在指定的这段时间之内保持新鲜</td></tr>
<tr><td>no-transform	</td><td>
请求
</td><td>文档在发送之前不允许被 转换</td></tr>
<tr><td>only-if-cached</td><td>
请求
</td><td>只有当文档在缓存中才发送，不要联系原始服务器</td></tr>
<tr><td>public	</td><td>
响应
</td><td>响应可以被任何服务器缓存</td></tr>
<tr><td>private</td><td>
响应
</td><td>响应可以被缓存，但只能被单个端访问</td></tr>
<tr><td>no-cache</td><td>
响应	
</td><td>如果指令伴随一个首部列表的话，那么内容可以被缓存并提供给客户端，但必须先删除所列出的首部，如果没有指定首部，缓存中的副本在没有重新向服务器验证之前不能提供给客户端</td></tr>
<tr><td>no-store</td><td>
响应
</td><td>响应不允许被缓存</td></tr>
<tr><td>must-revalidate</td><td>
响应
</td><td>响应在提供给客户端之前必须重新向服务器验证</td></tr>
<tr><td>proxy-revalidate</td><td>
响应
</td><td>共享的缓存在提供给客户端之前必须重新向原始服务器验证。私有的缓存可以忽略这条指令</td></tr>
<tr><td>max-age</td><td>
响应
</td><td>指定文档可以被缓存的时间以及新鲜度的最长时间</td></tr>
<tr><td>s-max-age</td><td>
响应
</td><td>指定文档作为共享缓存时的最长使用时间。私有的缓存可以忽略本指令</td></tr>

</table>
