* Host与server_name的匹配顺序是，严格字符串->前置通配符->后置通配符->正则。一个server可以监听多个server_name
* nginx进行location匹配的顺序:

    1. 如果uri和某一条location =严格匹配时，则命中这个location并且不再做别的匹配。
    2. 和所有的prefix进行匹配，记录下最长的匹配的。
    3. 如果最长的匹配的prefix前面有^~修饰符，则直接选取这个location。
    4. 按照配置文件顺序对所有正则进行匹配，选取第一个匹配的。如果全都没有匹配，则选取之前的那条已经选中的prefix的location。

* 明确rewrite的flag的有含义

    * 没有flag时，命中此条rewrite后，用rewritten_uri作为uri继续进行下面的rewrite。
    * last - 命中此条rewrite后，结束rewrite模块的所有指令，然后用rewritten_uri重新做一次location匹配。可以看到，这个的影响是严重的，因为命中后会重新搞一次location匹配，如果写的不好会引起死循环，nginx会在循环10次后返回500错误，因此last一般情况下不允许使用。
    * break - 命中此条rewrite后，结束rewrite模块的所有指令。
    * redirect - 命中此条rewrite后，返回302，Location响应头为rewritten_uri。事实上，rewrite的subject里面如果有http://或者https://开头的，所有都等于redirect。
    * permanent - 命中此条rewrite后，返回301，Location响应头为rewritten_uri。

    > rewrite的break和last都会导致处理流程跳出所有的rewrite模块的指令

* $arg_xxx，这里的xxx就是get参数中的xxx参数，相当于php的$_GET[‘xxx’]

    ```nginx
    rewrite ^/(mm)/?$ /$1/index.php/$arg_action break;
    ```

* nginx按照阶段顺序提取配置文件，并且在阶段内，按照配置顺序或者特定顺序执行

    ```nginx
    location / {
    proxy_set_header Host $req_host;  # content_phase阶段
    set $req_host m.baidu.com;        # rewrite阶段
    ...
    }
    ```

    如上面的配置片段，配置文件顺序是先出现的proxy_set_header后出现的set，那么按照这个指令顺序，在proxy_set_header的时候$req_host是个未初始化的变量，这一定会引起问题。但是事实上，这个配置并不影响功能。原因是，set是在rewrite阶段处理的，proxy_set_header是在content_phase阶段处理的，而content_phase在rewrite之后，所以这个配置并不影响功能。但是这样写很显然会引起人类阅读的困扰。

* 所有proxy_set_header必须放在server域，在http域内和location域内不得出现，如果必须在location域内使用这个，那么通过set设置变量来实现。