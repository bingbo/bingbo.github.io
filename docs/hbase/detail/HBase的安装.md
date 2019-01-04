1. 下载相应的安装包

    在这里下载`http://www.apache.org/dyn/closer.cgi/hbase/`或`http://apache.fayea.com/hbase/`直接选择相应的包下载，如`hbase-1.2.1-bin.tar.gz `

2. 解压配置
    
    ```bash
    tar xzvf hbase-1.2.1-bin.tar.gz
    cd hbase-1.2.1
    ##环境变量配置
    vim conf/hbase-env.sh
##添加以下环境变量
    export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_91.jdk/Contents/Home
    ```

    > 安装完java后可以通过以下方式查看环境变量
    ```bash
    $/usr/libexec/java_home
    /Library/Java/JavaVirtualMachines/jdk1.8.0_91.jdk/Contents/Home
    ```

3. 修改conf/hbase-site.xml添加数据路径

    ```xml
    <configuration>
    <property> 
    <name>hbase.rootdir</name> 
    <value>file:///Users/zhangbingbing/Work/hbase-1.2.1/data</value> 
    </property>
    </configuration>
    ```

4. 运行

    ```bash
    #启动服务
    bin/start-hbase.sh 
    ##进入shell命令行
    bin/hbase shell
    ##启动thrift服务，即可用php thrift客户端访问
    bin/hbase thrift -p 9090 start
    ```
    
